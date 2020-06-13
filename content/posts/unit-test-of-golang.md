---
title: "Unit test of Golang"
date: 2018-08-21T17:33:29+08:00
draft: false
categories:
  - tech
tags:
  - Golang
  - test
typora-root-url: ../../static
---

### notes

测试最重要的就是处理依赖, 覆盖所有的流程, 合理的 assertion.

我们在一个测试中, 不应对其依赖的实现做测试, 所以我们需要 mock 掉相应的实现. 由于 Golang 中 first class function 的存在, 我们可以通过在 package 中定义变量 (`var`) 的方式将依赖提到包级作用域中, 在测试中替换对应的逻辑, 从而不会对包中的执行代码产生影响.

依赖返回的类型主要有两种: 基本类型组合, interface

对于基本类型及其组合, 我们可以直接定义一个简单的工厂函数来生成他的返回值.

而如果依赖返回的是一个相对复杂的 interface, we need to implement an structure satisfy the interface.

### Links

- [Golang unit testing interfaces](https://blog.andreiavram.ro/golang-unit-testing-interfaces/)
- [MX record](https://en.wikipedia.org/wiki/MX_record)
- [First class function in Golang](https://blog.golang.org/first-class-functions-in-go-and-new-go)
