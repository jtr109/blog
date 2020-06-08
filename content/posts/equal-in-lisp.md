---
title: LISP 中的各种相等表示
date: 2017-11-28T07:52:09+08:00
tags:
  - LISP
typora-root-url: ../../static
---

## 简述

LISP 中描述相等的断言主要有 `eq`, `eql`, `equal` 和 `equalp`. 他们对应的判断条件各不相同.

## 逐个分析

### 断言 `eq`

`(eq x y)` 仅当 `x` 和 `y` 是同一对象时才返回 `true`
