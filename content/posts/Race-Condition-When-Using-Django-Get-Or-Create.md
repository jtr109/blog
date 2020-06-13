---
title: Django `get_or_create` 创建重复条目问题分析和处理
date: 2019-11-09 22:09:45
categories:
  - tech
tags:
  - Python
  - Django
typora-root-url: ../../static
---

## 名词解释

* Data Race & Race Condition: 待补充, 可参考[这个 StackOverflow 回答](https://stackoverflow.com/a/58161437/6522746)

## 问题描述

Django 的 ORM 操作中有一个非常实用的功能: [`get_or_create`](https://github.com/django/django/blob/2.2.6/django/db/models/query.py#L528), 可以判断数据库中是否有符合条件的条目, 没有则创建.

但是在使用过程中如果不加留意, 会发生 race condition. 导致数据库中创建出的条目不符合预期.

以下面这部分代码为例:

```python
# models.py

class Race(models.Model):
    user_id = models.CharField(max_length=32)
    name = models.CharField(max_length=32)
```

如果在项目中调用遇到了并发的情况, 会导致数据库中有一条以上的相同条目. 可以编写单元测试复现该问题:

```python
# tests.py

class RaceTestCase(TestCase):

    def test_race(self):

        def get_or_create():
            Race.objects.get_or_create(
                user_id='c53b8ba212ef4f15b2365e1bb3d524fe',
                defaults=dict(name='foo', age=1),
            )

        threads = list()
        for _ in range(10):
            threads.append(threading.Thread(target=get_or_create))
        for t in threads:
            t.start()
        for t in threads:
            t.join()
        self.assertNotEqual(Race.objects.count(), 1)
```

如果执行测试, 有一定概率会返回如下信息:

```shell
$ python manage.py test
...
Exception in thread Thread-7:
Traceback (most recent call last):
  File "/Users/jtr109/.pyenv/versions/3.7.4/lib/python3.7/threading.py", line 926, in _bootstrap_inner
    self.run()
  File "/Users/jtr109/.pyenv/versions/3.7.4/lib/python3.7/threading.py", line 870, in run
    self._target(*self._args, **self._kwargs)
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/race/tests.py", line 20, in get_or_create
    defaults=dict(name='foo', age=1),
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/venv/lib/python3.7/site-packages/django/db/models/manager.py", line 82, in manager_method
    return getattr(self.get_queryset(), name)(*args, **kwargs)
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/venv/lib/python3.7/site-packages/django/db/models/query.py", line 538, in get_or_create
    return self.get(**kwargs), False
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/venv/lib/python3.7/site-packages/django/db/models/query.py", line 412, in get
    (self.model._meta.object_name, num)
race.models.Race.MultipleObjectsReturned: get() returned more than one Race -- it returned 2!

Exception in thread Thread-9:
Traceback (most recent call last):
  File "/Users/jtr109/.pyenv/versions/3.7.4/lib/python3.7/threading.py", line 926, in _bootstrap_inner
    self.run()
  File "/Users/jtr109/.pyenv/versions/3.7.4/lib/python3.7/threading.py", line 870, in run
    self._target(*self._args, **self._kwargs)
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/race/tests.py", line 20, in get_or_create
    defaults=dict(name='foo', age=1),
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/venv/lib/python3.7/site-packages/django/db/models/manager.py", line 82, in manager_method
    return getattr(self.get_queryset(), name)(*args, **kwargs)
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/venv/lib/python3.7/site-packages/django/db/models/query.py", line 538, in get_or_create
    return self.get(**kwargs), False
  File "/Users/jtr109/mystuff/learn/django-update-or-create-example/venv/lib/python3.7/site-packages/django/db/models/query.py", line 412, in get
    (self.model._meta.object_name, num)
race.models.Race.MultipleObjectsReturned: get() returned more than one Race -- it returned 2!

.
----------------------------------------------------------------------
Ran 1 test in 0.107s

OK
Destroying test database for alias 'default'...
```

_由于使用 `threading` 实现并发, 所以不一定每次执行都会出现如上状况. 如果没有复现, 请重试几次._

从上面的信息可以看到, 数据库中生成了不止一个条目, 而且有的 threading 在执行时报 `get() returned more than one Race` 错误.

## 原因分析

查看 [`get_or_create`](https://github.com/django/django/blob/2.2.6/django/db/models/query.py#L528) 源码可以看到, 这个操作分为两步: 查询和创建. 所以在异步场景下, 可能出现多个读操作都没有查到条目, 触发 `DoesNotExist` 报错, 写入多条, 而非预期中的始终只有一条.

```python
def get_or_create(self, defaults=None, **kwargs):
    """
    Look up an object with the given kwargs, creating one if necessary.
    Return a tuple of (object, created), where created is a boolean
    specifying whether an object was created.
    """
    # The get() needs to be targeted at the write database in order
    # to avoid potential transaction consistency problems.
    self._for_write = True
    try:
        return self.get(**kwargs), False
    except self.model.DoesNotExist:
        params = self._extract_model_params(defaults, **kwargs)
        return self._create_object_from_params(kwargs, params)
```

## 解决方案

这个问题其实在文档中[已经指出](https://docs.djangoproject.com/en/dev/ref/models/querysets/#get-or-create):

> This method is atomic assuming that the database enforces uniqueness of the keyword arguments (see [`unique`](https://docs.djangoproject.com/en/dev/ref/models/fields/#django.db.models.Field.unique) or [`unique_together`](https://docs.djangoproject.com/en/dev/ref/models/options/#django.db.models.Options.unique_together)). If the fields used in the keyword arguments do not have a uniqueness constraint, concurrent calls to this method may result in multiple rows with the same parameters being inserted.

所以如果希望避免条目冲突, 需要将 `get_or_create` 参照的列设置为 unique. 对于多列参照的情况, 必须保证 `unique_together`.

以 `Race`为例进行改造:

```python
# models.py

class Solution(models.Model):
    user_id = models.CharField(max_length=32, unique=True)  # declaring `user_id` uniqueness
    name = models.CharField(max_length=32)
```

补充一个 `unique_together` 示例:

```python
# models.py

class Together(models.Model):
    user_id = models.CharField(max_length=32)
    name = models.CharField(max_length=32)
    age = models.PositiveIntegerField()

    class Meta:
        unique_together = [
            # declaring the combination of `user_id` and `name` is unique
            ['user_id', 'name'],
        ]

```

完整示例可以参考[示例项目](https://github.com/jtr109/django-update-or-create-example)

## 源码分析

在源码中是如何使用 `translation.atom` 解决这个错误的, 待后续分析.
