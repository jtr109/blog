---
title: "Python 给形参定义随机默认值的注意事项"
date: 2021-01-07T11:15:44+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Python
---

## 概览

我们在定义形参时会使用默认值，但是需要注意这些默认值在初始化的时候就会被实例化。所以不应该在定义形参的时候使用随机函数指定默认值。

## 问题描述

如果我们定义如下代码：

```python
import uuid

def foo(bar=uuid.uuid4()):
    return bar
```

执行上述代码时，如果不指定参数值，会发现每次的返回值并不是随机的。

```python
foo()  # UUID('21b425cf-c9a6-4479-95e7-061ebed36511')
foo()  # UUID('21b425cf-c9a6-4479-95e7-061ebed36511')
foo()  # UUID('21b425cf-c9a6-4479-95e7-061ebed36511')
```

如果我们定义类方法，也会遇到相同的问题。

```python
import uuid
import random

class Foo:
  
    def __init__(self, bar=uuid.uuid4()):
        self.bar = bar
        
    def biz(self, buz=random.randint(1, 10)):
        return buz
```

## 解决方案

所以在如果希望定义一个随机的形参，必须在函数内部定义。

```python
def foo(bar=None):
    if bar is None:
        return uuid.uuid4()
    return bar
```

