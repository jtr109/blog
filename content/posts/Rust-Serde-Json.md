---
title: Rust Serde Json
categories:
  - tech
date: 2019-12-24 23:34:10
tags:
  - Rust
typora-root-url: ../../static
---

## 背景

今天学习 The Rust Cookbook 的时候看到一段 JSON 解码的代码:

```rust
#[macro_use]
extern crate serde_json;

use serde_json::{Value, Error};

fn main() -> Result<(), Error> {
    let j = r#"{
                 "userid": 103609,
                 "verified": true,
                 "access_privileges": [
                   "user",
                   "admin"
                 ]
               }"#;

    let parsed: Value = serde_json::from_str(j)?;

    let expected = json!({
        "userid": 103609,
        "verified": true,
        "access_privileges": [
            "user",
            "admin"
        ]
    });

    assert_eq!(parsed, expected);

    Ok(())
}
```

## 疑问

这里衍生出一个问题: "变量 `expected` 被一个宏 `json!` 的结果赋值, 那这个 `expected` 到底是什么类型?"

查看[文档](https://docs.serde.rs/serde_json/value/enum.Value.html)可以了解到这是一个 `serde_json::value::Value`.

那随之而来的就是另一个问题: 怎么把 `Value` 转换为实际类型.

举个例子, `expected` 中的 `userid` 的 `u64` 版本应该怎么获取?

<!-- more -->

## 提取字段值

文档中说明了可以显式指定转换类型, 生成对应类型的值, 例如 `T`: `Option<T>`

```rust
assert_eq!(expected["userid"].as_u64().unwrap(), 103609);
```

## 转换为结构体

虽然对单一字段可以这样操作, 但是将读取的 json 字符串转换为一个结构体是更合适的选择:

```rust
extern crate serde_json;

use serde::Deserialize;
use serde_json::Error;

#[derive(Debug, Deserialize)]
struct User {
    userid: u64,
    verified: bool,
    access_privileges: Vec<String>,
}

fn main() -> Result<(), Error> {
    let j = r#"{
        "userid": 103609,
        "verified": true,
        "access_privileges": [
            "user",
            "admin"
        ]
    }"#;

    let parsed: User = serde_json::from_str(j)?;

    assert_eq!(parsed.userid, 103609);

    Ok(())
}
```





