---
title: 查看 Rust Crates 的 Features
date: 2021-05-29T16:24:18+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Rust
  - Cargo
---

## 背景

在 Rust 学习过程中，我发现很多库的示例中会制定 `features` flag，我理解 features 是用来指定使用库的部分功能。例如 Tokio 的文档说明：

> Tokio has a lot of functionality (TCP, UDP, Unix sockets, timers, sync utilities, multiple scheduler types, etc). Not all applications need all functionality. When attempting to optimize compile time or the end application footprint, the application can decide to opt into **only** the features it uses.

[原文链接](https://tokio.rs/tokio/tutorial/hello-tokio#:~:text=tokio%20has%20a%20lot%20of%20functionality%20(tcp%2C%20udp%2C%20unix%20sockets%2C%20timers%2C%20sync%20utilities%2C%20multiple%20scheduler%20types%2C%20etc).%20not%20all%20applications%20need%20all%20functionality.%20when%20attempting%20to%20optimize%20compile%20time%20or%20the%20end%20application%20footprint%2C%20the%20application%20can%20decide%20to%20opt%20into%20only%20the%20features%20it%20uses.)

## 查看可选的 Features

但是作为一个使用者我应该怎么知道一个库有哪些 features 可以选择，这些 features 分别代表什么含义呢？

在 Tokio 的 docs.rs [文档](https://docs.rs/tokio/1.6.1/tokio/) 上，我发现了最上方导航栏中的 [`Feature flags`](https://docs.rs/crate/tokio/1.6.1/features) 栏，点击即可查看到可选的 features 的选项以及这些选项提供哪些功能的信息。

## 声明可选的 Features

那下一个疑问就是作为 crate 作者应该怎样声明支持的 features 呢？查看 The Cargo Book 中的[解释](https://doc.rust-lang.org/cargo/reference/features.html#the-features-section)，是在库的 [`Cargo.toml`](https://doc.rust-lang.org/cargo/reference/features.html#the-features-section) 中通过 `[features]` 段落指定的。

另外也[建议](https://doc.rust-lang.org/cargo/reference/features.html#:~:text=you%20are%20encouraged%20to%20document%20which%20features%20are%20available%20in%20your%20package.%20this%20can%20be%20done%20by%20adding%20doc%20comments%20at%20the%20top%20of%20lib.rs.) crate 的作者在文档中解释 features 的用法：

> You are encouraged to document which features are available in your package. This can be done by adding [doc comments](https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html) at the top of `lib.rs`.

