---
title: "Antfin Interview"
date: 2018-09-12T21:51:16+08:00
draft: true
categories:
  - tech
tags:
  - interview
typora-root-url: ../../static
---

## Antfin phone interview

_今天参加了蚂蚁金服的电话面试, 自己实力不济, 加之没想到晚上会突然打电话过来, 几乎没有准备, 表现非常不理想. 不过还是把面试题目和 review 的答案整理出来, 也算是一点收获._

### 二维数组查找

问题: 一个二维数组, 每行递增, 每列递减. 如何定位一个值.

答案:

```python
a = [
    [4, 8, 12, 16],
    [3, 7, 11, 15],
    [2, 6, 10, 14],
    [1, 5, 9,  13],
]

def get(arr, target):
    # TODO: 这里没有考虑该值不存在的情况
    r, c = 0, 0
    while True:
        guess = arr[r][c]
        if guess == target:
            return (r, c)
        elif guess < target:
            c += 1
        else:
            r += 1

get(a, 5)  # (3, 1)
```

### 合并两个有序数组

问题: 如何合并两个有序数组

答案:

```python
def merge(l1, l2):
    tmp = list()
    i, j = 0, 0
    while i < len(l1) and j < len(l2):
        if l1[i] < l2[j]:
            tmp.append(l1[i])
            i += 1
        elif l1[i] > l2[j]:
            tmp.append(l2[j])
            j += 1
        else:
            tmp.extend((l1[i], l2[j]))
            i += 1
            j += 1
    tmp.extend(l1[i:])
    tmp.extend(l2[j:])
    return tmp
```

进一步问: 如果其中一个 list 足够放下两个数组, 不借助第三个数组排序, 要求复杂度为常量.

例如:

```python
l1 = [1, 3, 5, None, None, None]
l2 = [2, 4]
```

个人回答靠右排序在第一个数组中. 应该不是正确答案, 稍后求证.


### GIL 锁的意义

参见[这篇文章](https://github.com/taizilongxu/interview_python#18-gil%E7%BA%BF%E7%A8%8B%E5%85%A8%E5%B1%80%E9%94%81)

- Python 为了保证线程安全而采取的独立线程运行的限制,说白了就是一个核只能在同一时间运行一个线程.
- 对于io密集型任务, python的多线程起到作用, 但对于cpu密集型任务, python的多线程几乎占不到任何优势, 还有可能因为争夺资源而变慢.

追问: coroutine 为什么能解决 GIL 的性能问题?

回答:

GIL 的大致逻辑是使用系统自身的分时复用机制实现并发. GIL 能解决的问题是 wait simulaneously, 即 IO 等待时间过长, 会切换执行其他线程. 但是如果对于计算量大, 每个线程有比较大的变量存在时, GIL 的机制会导致计算到一半的线程被暂停, 如内存等资源的竞争, context 的切换成本都对性能造成非常大的影响. 导致基于 GIL 的并发使用场景并不广泛.

而 coroutine 的好处在于把切换的时机交给协程自己, 使协程在合适的时机被暂停和上下文切换.

简而言之:

- GIL 在 IO 密集型的场景中没有太大性能问题
- 但是在计算密集型的场景中会由于 context switching 的问题导致性能大打折扣
- coroutine stack 由解释器自身维护, 可以更好地掌握切换时机

参考文章: [Working simultanously vs waiting simultaneously](http://yosefk.com/blog/working-simultaneously-vs-waiting-simultaneously.html)

### 哪些字段加 db index?

- 简要的索引介绍在[这里](https://www.jianshu.com/p/4c2a2b0ef3e0)
- 延伸阅读可以查看[这篇文章](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

- 如果一个字段频繁出现在 WHERE 子句中, 那么它应该被添加索引
- 尽量使用唯一索引而非全文索引
- 索引可以加快查询的速度, 但是在表的数据变化时, 索引的动态维护会降低速度

### 查询数据库的时候哪些字段放在前面?

- 优先较少结果
- 优先 index
- 优先 `=`

### iterable 和 iterator 的区别

关于 iterator 和 iterable 的区别在 [python wiki](https://wiki.python.org/moin/Iterator) 中有解答:

> An **iterable** object is an object that implements `__iter__`, which is expected to return an **iterator** object.
> An **iterator** is an object that implements `next`, which is expected to return the next element of the iterable object that returned it, and raise a `StopIteration` exception when no more elements are available.

我们以文章中的示例来 review 一下:

```python
import random

class RandomIterable(object):
    def __iter__(self):
        return self
    def __next__(self):
        if random.choice(['go', 'go', 'stop']) == 'stop':
            raise StopIteraion
        return 1

r = RandomIterable()
next(r)
```

这里的 `RandomIterable` 就是一个 iterable. 他的 `__iter__` 方法返回了自身的实例 `r`, 这个实例就是一个 iterator. 实例必须定义了 `__next__` 方法. 调用内置函数 `next` 就可以获取下一个返回值.

> An object isn't iterable unless it provides __iter__. And for an object to be a valid iterator, it must provide next.

### 扩展思考: generator 的定义

关于 generator 的定义在[这里](https://wiki.python.org/moin/Generators)可以了解:

> Generator functions allow you to declare a function that behaves like an iterator, i.e. it can be used in a for loop.

看了这句话应该很明显了, `range` 就是一个典型的 generator.

推荐使用 `yield` 实现, 例如:

```python
def yrange(n):
    i = 0
    while i < n:
        yield i
        i += 1
```

Reference in [Python Practic Book](https://anandology.com/python-practice-book/iterators.html).

### http 请求结构

参考[网页](http://www.runoob.com/http/http-messages.html)

客户端发送一个 HTTP 请求到服务器的请求消息包括以下格式: 请求行（request line）, 请求头部（header）, 空行和请求数据四个部分组成.

补充: HTTP 响应也由四个部分组成，分别是: 状态行, 消息报头, 空行和响应正文.

### 数据库缓存的方式

- redis 缓存会遇到的问题及解决方法可以参考[这篇文章](https://juejin.im/post/5b961172f265da0ab7198f4d)

### redis 缓存数据库数据的结构

可以参考[这篇文章](https://blog.csdn.net/qtyl1988/article/details/39519951)学习一下.

redis 的主要数据结构有:

- hash
- string
- list
- set
- sorted set

数据库的结构归根结底是以行存储的, 所以适合的结构是 string and hash.

一个比较直观的想法, 就是将数据转化为 json 格式, 以 string 结构存储在 redis 中.

#### 延伸问题: 每条数据的 key 如何指定

可以通过查询语句的内容作为 key, 但是不应该直接使用该字符串, 一个显而易见的原因是 SQL 中有空格. 可以使用 MD5 等方式生成唯一标识符.
