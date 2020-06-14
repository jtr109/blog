---
title: "use foo::bar::* 导致 mismatched types 报错"
date: 2019-08-13T11:13:09+08:00
typora-root-url: ../../static
draft: false
categories:
  - tech
series:
  - Rust Notes
tags:
  - Rust
  - module
  - Diesel
---

## 问题描述

今天跟着[文章](https://dev.to/werner/practical-rust-web-development-api-rest-29g1)学习的时候遇到一个问题. 我把这句导入语句

```rust
use crate::schema::products::dsl::*;
```

放在了顶部,  在下面定义了一个 method:

```rust
impl Product {
    pub fn find(id: &i32) -> Result<Product, diesel::result::Error> {
        let connection = establish_connection();

        products::table.find(id).first(&connection)
    }
}
```

结果编译报错:

```shell
error[E0308]: mismatched types
  --> src/models/product.rs:17:17
   |
17 |     pub fn find(id: &i32) -> Result<Product, diesel::result::Error> {
   |                 ^^ expected i32, found struct `schema::products::columns::id`
   |
   = note: expected type `i32`
              found type `schema::products::columns::id`
```


## 原因

原因出在 `use foo::bar::*` 的时候, 编译器将 struct `id` 引入了环境. 然后在 method 定义参数的时候, 编译器认为这里的 `id` 指的是一个 pattern. 所以报错.

举一个类似的例子:

```rust
struct Foo;

fn bar(Foo: i32) {}

main() {}
```

以上例子中, 函数 `bar` 里定义的 `Foo` 被编译器认为是一个 pattern, 所以会提示 mismatched types.


## 解决方案

这里的主要原因是 `diesel`  的变量名设计存在一些问题. 这里其实用 CamelCase 会更合适.

另外应该避免使用 `use foo::bar::*` 这样的语法.

那如果不得不用的时候, 特别是针对 `diesel` 使用的时候, 可以尽量考虑在函数内调用 `use`.
