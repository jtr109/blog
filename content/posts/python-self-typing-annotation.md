---
title: "Python Class 定义时声明当前 Class 类型"
date: 2020-10-23T11:10:55+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Python
  - mypy
---

## 问题描述

Typing annotation 可以帮助我们更好地构建代码，但是由于在 Python 中这是一个新生事物，对它的原生支持做的并不好。

_声明：一下所有场景基于 Python 3.8.2。_

让我们一起来看看下面这个例子：

```python
class Foo:

    def __init__(self: Foo, bar: str) -> None:
        self.bar = bar

if __name__ == '__main__':
    Foo('bar')
```

该例子中我们按常规方式声明了 `self` 的类型 `Foo`。让我们尝试验证类型和执行代码。

```shell
$ mypy main.py
Success: no issues found in 1 source file

$ python main.py
Traceback (most recent call last):
  File "main.py", line 1, in <module>
    class Foo:
  File "main.py", line 3, in Foo
    def __init__(self: Foo, bar: str) -> None:
NameError: name 'Foo' is not defined
```

因为生命方式符合 [mypy](http://mypy-lang.org/) 的声明标准，所以成功通过了。但是执行时发生了报错，因为在类 `Foo` 定义前就已经被自己的方法 `__init__` 使用了，解释器无法理解，报出 `NameError: name 'Foo' is not defined` 错误。

## 解决方案

我们需要一个生命方法，可以同时满足 `mypy` 的类型生命校验，也可以满足解释器的运行规则。文章汇总了几种解决方案：

### Forward References

[Forward references](https://www.python.org/dev/peps/pep-0484/#forward-references) 方法出自 [PEP-484](https://www.python.org/dev/peps/pep-0484/)。

> When a type hint contains names that have not been defined yet, that definition may be expressed as a string literal, to be resolved later.

我们可以使用 `"Foo"` 字符串作为声明的类型。

```python
class Foo:

    def __init__(self: 'Foo', bar: str) -> None:
        self.bar = bar
```

### Postponed Evaluation of Annotations

[PEP-563](https://www.python.org/dev/peps/pep-0563/) 介绍了一个 `__future__` 功能。使我们可以更符合直觉地使用类型声明。

>  [PEP 484](https://www.python.org/dev/peps/pep-0484) introduced a standard meaning to annotations: type hints. [PEP 526](https://www.python.org/dev/peps/pep-0526) defined variable annotations, explicitly tying them with the type hinting use case.

我们需要在模块中导入 `__annotation__`。

```python
from __future__ import annotations

class Foo:

    def __init__(self: Foo, bar: str) -> None:
        self.bar = bar
```

