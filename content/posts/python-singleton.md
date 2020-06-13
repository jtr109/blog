---
title: "Python Singleton"
date: 2018-09-07T07:34:23+08:00
draft: false
typora-root-url: ../../static
categories:
  - tech
tags:
  - Python
  - pattern
---

## Ways to use singleton in Python

1. using module
2. using method `__new__`
3. using decorator
4. using metaclass

## Analysis the singleton with method `__new__`

### Implementation

Let us see the code first:

```python
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kw):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kw)
        return cls._instance
```

And check the result:

```python
>>> s1 = Singleton()
>>> s2 = Singleton()

>>> s1 is s2
True

>>> print(id(s1), id(s2))
4420874480 4420874480
```

### How does it work?

We should know two feature to understand that:

1. **Class variables** are shared by all instances of the class. (Learn more in [document](https://docs.python.org/2/tutorial/classes.html#class-and-instance-variables).)
2. The `__new__` method **return a instance** of class which will be used as `self` in `__init__` and other methods. (Learn more in [document](https://docs.python.org/3/reference/datamodel.html#object.__new__))

### Here we go

When declaring a new instance, the [`__new__`](https://docs.python.org/3/reference/datamodel.html#object.__new__) method check the existance of the class variable `_instance`. Return it if exists. Otherwise, the class creates a new one, assigns to `_instance` and return.

After that, all instances of the class `Singleton` are the same class variable of class `Singleton`.

## Implement with decorator

### Implementation

```python
def singleton(cls):
    _instances = dict()
    def wrapper(*args, **kwargs):
        if cls not in _instances:
            _instances[cls] = cls(*args, **kwargs)
        return _instances[cls]
    return wrapper
```

Let us check the result

```python
@singleton
class MyClass(object):
    pass

s1 = MyClass()
s2 = MyClass()

s1 is s2  # True
```

### How it works?

- The criticle point is the implementation of **closure** in python. The variable `_instances` above did not collected by GC after `MyClass` was called. (You can learn more [here](https://www.geeksforgeeks.org/python-closures/))


## Analysis the singleton with method **metaclass**

### Implementation

```python
class Singleton(type):
    _instances = dict()
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]
```

Check the result

```python
class MyClass(metaclass=Singleton):  # in Python 3
    pass

# class MyClass(object):  # in Python 2
#     __metaclass__ = Singleton

s1 = MyClass()
s2 = MyClass()

s1 is s2  # True
```

### How it works?

- All class is an instance of a metaclass.
- When the class is called it return the result which the `__call__` method of metaclass return. (If not be overrided.)
- So we return the same instance by means of the shared class variable `_instances` in metaclass `Singleton` to return a same instance.
- By the way, the parent of metaclass should be the class `type`.

## Reference

- [Python 中的单例模式](https://segmentfault.com/a/1190000008141049)
