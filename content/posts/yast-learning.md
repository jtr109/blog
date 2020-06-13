---
title: Yet Another Scheme Tutorial 学习笔记
date: 2017-08-23T07:30:11+08:00
categories:
  - Tech
tags:
  - Scheme
  - LISP
  - SICP
typora-root-url: ../../static
---

## 算数操作

### 基本算数操作

- 函数 `exact->inexact` 用于把分数转换为浮点数.
- 函数 `quotient` 用于求商数（quotient）。
- 函数 `remainder` 和 `modulo` 用于求余数（remainder）。
- 函数 `sqrt` 用于求参数的平方根（square root）。

```LISP
(exact->inexact (/ 29 3 7))  ;-> 1.380952380952381
(quotient 7 3)  ;→ 2
(modulo 7 3)  ;→ 1
(sqrt 8)  ;→ 2.8284271247461903
```

### 练习

练习中发现, 如果创建 scm 文件并写入多个算式, 好像不能得出每个计算的值, 得到的只能是最后一个算式的结果.

## 表

### 空表

`'()` 即为空表

有两种非常类似的 value, 实现的方式却是不同的:

- `(3 2 . 1)`:
  ![`(3 2 .1)`](http://www.shido.info/lisp/conss2.png)
- `(1 2 3)`:
  ![`(1 2 3)`](http://www.shido.info/lisp/list2.png)

其实主要的差异是最后一个 cons 的 cdr 指向地址的不同, 上面一个是指向 1 所在的地址, 最后一个是指向一个空表 `'()`

这里指出一下[中文版](https://deathking.github.io/yast-cn/contents/chapter3.html)的图片有误, 请参照[原版](http://www.shido.info/lisp/scheme3_e.html)

## 引用

`'()`这里的空表就是使用了引用(quote)

## list 函数

特别值得注意的是, list 函数返回的是其中的元素构成的表, 函数中的每个参数都是表的一个元素:

```Scheme
(list '(1 2) '(3 4))
;Value 1: ((1 2) (3 4))

(cons '(1 2) '(3 4))
;Value 2: ((1 2) 3 4)
```

## 字符串操作

- `string-append`: 字符串拼接

## 函数定义

### 定义变量

```Scheme
(define vhello "Hello world!")
```

### 定义函数

定义函数有两种方式, 一个是 lambda 一个是简写, 暂时不清楚两者的实际区别.

```Scheme
;;; 定义不带参数的函数

; lambda 定义
(define hello
  (lambda ()
    "Hello world!"))

; 简写
; (define (hello)
;  ("Hello world!"))
; 不成立

;;; 定义带参数的函数

; lambda
(define hello
  (lambda (name)
    (string-append "Hello " name "!")))

; 简写
(define (hello name)
  (string-append "Hello " name "!"))

```

## if 判断

### true and false

在 LISP 中 true 用 `#t` 表示, false 用 `#f` 表示.

### 关于空表的判断

由于 R4RS 和 R5RS 对空表 `'()` 是否为 false (`#f`) 的标准不同, 所以判断表为空应使用 `null?`:

```Scheme
(null? '())
;Value: #t
```

### 数字判断

- `positive?`: 判断正数
- `zero?`: 是否为 0
- `negative?`: 判断负数
- `odd?`: 奇数
- `even?`: 偶数

### 原生函数

- `integer->char`: 正数转化为 ascii 码

## and 和 or

Scheme 中的 and 和 or 相对 C 类语言来说十分特殊:

> `and` 具有任意个数的参数，并从左到右对它们求值。
> 如果某一参数为 `#f`，那么它就返回 `#f`，而不对剩余参数求值。
> 反过来说，如果所有的参数都不是 `#f`，那么就返回最后一个参数的值。

> `or` 具有可变个数的参数，并从左到右对它们求值。
> 它返回第一个不是值 `#f` 的参数，而余下的参数不会被求值。
> 如果所有的参数的值都是 `#f` 的话，则返回最后一个参数的值。

关于 `or` 逻辑的最后依据可以看作 or 会从左往右求职到第一个非 `#f` 的值, 如果没有 `#f` 会一直计算到最后一个参数, 并返回其结果 `#f`


## 判断函数


### `eq?`, `eqv?` and `equal?`

- `eq?` 用于判断两个对象的地址,  适合用来比较两个变量.
- `eqv?` 比较两个存储在内存中的对象的类型和值, 如果类型和值都一致的话就返回 `#t`. 适合用来比较数值, 但是 int 和 float 比较的返回为 `#f`
- `equal?` 用于比较类似于表或者字符串一类的序列

## 单一变量的判断

- `pair?`: 如果对象为序对则返回#t
- `list?`: 如果对象是一个表则返回#t。要小心的是空表’()是一个表但是不是一个序对
- `null?`: 如果对象是空表’()的话就返回#t
- `symbol?`: 如果对象是一个符号则返回#t
- `char?`: 如果对象是一个字符则返回#t
- `string?`: 如果对象是一个字符串则返回#t
- `number?`: 如果对象是一个数字则返回#t
- `complex?`: 如果对象是一个复数则返回#t
- `real?`: 如果对象是一个实数则返回#t
- `rational?`: 如果对象是一个有理数则返回#t
- `integer?`: 如果对象是一个整数则返回#t
- `exact?`: 如果对象不是一个浮点数的话则返回#t
- `inexact?`: 如果对象是一个浮点数的话则返回#t

## 字符比较

`char=?`, `char<?`, `char>?`, `char<=?`, `char>=?`

## 比较字符串

`string=?`, `string-cli=?`

## 关于循环和 lambda

[第七章: 重复](https://deathking.github.io/yast-cn/contents/chapter7.html)(或[英文版](http://www.shido.info/lisp/scheme7_e.html))中有这样一道题:

> 一个分别接受一个表 `ls` 和一个对象 `x` 的函数，该函数返回从 `ls` 中删除 `x` 后得到的表。

后面给出了习题答案, 但是处理 "过于优雅", 对初学者不是很友好, 在[这个回答](https://stackoverflow.com/questions/1905222/how-to-delete-an-element-from-a-list-in-scheme#answer-15017744)中可以找到更易懂的解题方法. 下面把两种做一个对照:

<script src="https://gist.github.com/jtr109/3f8577f07255af1e2b3d9b3fd09e8774.js"></script>

### 浅谈 lambda

原答案中最让人纠结的地方就是 if... lambda... 的组合使用, 由于之前本课程中没有过对于 lambda 的介绍, 只有 define 函数的时候提到过, 所以难免不清楚怎么使用. 其实可以在终端中动手试试, 你会发现 lambda 的用法如下:

```Scheme
((lambda (x) (* 2 x)) 1)
;Value: 2

((lambda (x y) (* 2 x (+ 1 y))) 2 3)
;Value: 16
```

以上就是定义 lambda 匿名函数的方法, 在最外层的括号中有两个组成部分, 一个是定义 lambda 匿名函数的括号, 还有一组参数. 再对应到原书中的答案, 就可以发现原答案中是根据判断定义匿名函数, 并作用在 `remove x (cdr ls)` 上. 这样考虑就简单了许多.

## named let and letrec

named let 用于迭代, 而 letrec 用于递归.

在 named let 中使用 `let` 初始化变量, 并在每次执行到命名的函数时循环. `letrec` 允许递归调用本身.

从实践来看, named let 解决了 while 类循环在 scheme 中的实现, 但是 `letrec` 感觉只是为函数定义限制了作用域.


## 映射

```Scheme
(map procedure list1 list2 ...)
```

`procedure` 是个与某个过程或 lambda 表达式相绑定的符号。作为参数的表的个数视 `procedure` 需要的参数而定。


## Reduce

关于 reduce 例子, 书里说的十分含糊, 经过资料查找, 可以发现示例来自[GNU MIT-Scheme](https://www.gnu.org/software/mit-scheme/documentation/mit-scheme-ref/Reduction-of-Lists.html)的官方文档, 这里可以了解到:

> The argument initial is used only if list is empty;


## Apply 函数

这个函数类似于普通语言中对函数的使用方法, 由 `apply`, 固定数量的参数, 和不固定参数构成的表组成, 类似于 Python 中的 `func(*args, **kwargs)`

## 词法闭包

作用域与源代码书写方式一致的作用域称为“词法闭包（Lexical closure）”或“静态作用域（Static scope）”。

## 关联表

函数`assq`，`assv`，和`assoc`从关联表中搜寻一个项。这些函数从开始一步步搜索关联表。如果它们找到序对的`car`等于给定的`key`，就返回该序对。如果找不到函数返回`#f`。这些函数分别使用`eq?`，`eqv?`，和`equal?`比较键，这意味着`assq`最快，`assoc`最慢。
