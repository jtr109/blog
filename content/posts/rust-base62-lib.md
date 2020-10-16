---
title: "Rust Base62 库学习和分析"
date: 2020-09-15T16:02:04+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Rust Notes
tags:
  - Rust
---

## 背景

在学习设计 Short URL 时尝试基于 Rust 编写了一个 Base62 库 [base62num](https://crates.io/crates/base62num)。后来发现已经有一个 Rust 库 [base-62](https://crates.io/crates/base-62) 提供了 Base62 的功能。

## 差异分析

因此我尝试对比 base-62 和自己项目的源码和功能做了一次学习。

#### 字符集映射

base-62 使用的字符集是：

```
0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
```

而 base62num 中使用的是：

```
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789
```

我在构建 base62num 时纠结过 alphanumeric 的顺序，但是没有找到明确的标准说明顺序。只有 Wikipedia 中明确给出了[字符映射表](https://en.wikipedia.org/wiki/Base62#Base62_table)，所以我选择了后者。

#### 正整型的形式

开始设计时，我只考虑了 32 bit 的 MurmurHash，所以认为 `usize` 已经可以满足需求。但实际上 `usize` 的大小对于 Base62 的使用场景来说还是不太够用的。

我们来看看 base-62 中的设计思路。在 base-62 中 [`encode`](https://docs.rs/base-62/0.1.1/base_62/base62/fn.encode.html) 的参数是 bytes：

```rust
pub fn encode(bytes: &[u8]) -> String
```

文档中是这样描述这个参数 bytes 的定义的：

> A byte array (leading zeros allowed) is prepended with `0x01` and is treated as a big-endian unsigned integer (`num_bigint::BigUint`).

这样带来了一个好处：该库的参数不受类型定义限制。因为无论是标准库[^1]还是 `num_bigint` [^2]都实现了对 byte array 的转化。所以将 byte array 避免了适应不同的类型，提高了通用性。这一点非常值得学习。

### 实现差异

#### 常量定义

base-62 [定义字母表常量](https://docs.rs/base-62/0.1.1/src/base_62/base62.rs.html#54)时使用的类型是 `[char; 62]`，而非 `String`。这样可以避免反复执行 `String::chars` 的 CPU 消耗。

#### 错误处理

base-62 在定义错误类型时使用了 `failure` 库，降低了代码复杂度。是一个不错的方法。

## 未来计划

一开始没有使用 base-62 的原因是没有理解为什么使用了 byte array 这样的方式作为参数。目前 base-62 对我而言美中不足的就是字母表顺序和我预期不一致。未来会考虑将自己的项目存档，通过 `features` 给 base-62 提供可选的其他字母表顺序的支持。

[^1]: 以 `usize` 为例，提供了 [`to_be_bytes`](https://doc.rust-lang.org/std/primitive.usize.html#method.to_be_bytes) 方法。`u32`, `u64` 等也均有此方法。
[^2]:  `num_bigint` 也实现了 byte array 转换，但是使用了的名称 [to_bytes_be](https://docs.rs/num-bigint/0.2.3/num_bigint/struct.BigUint.html#method.to_bytes_be)，和标准库中（如 `std::usize::to_be_types`）不一致，需留意。

