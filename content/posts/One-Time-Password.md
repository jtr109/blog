---
title: 一次性密码
categories:
  - Tech
date: 2020-01-07 23:45:24
tags:
  - security
  - Rust
typora-root-url: ../../static
---

## 背景

很多大型网站都提供了一种动态数字密码作为两步认证（2FA）的方案。你只要通过常用的密码管理器（例如 1Password，Authy 等）扫描二维码，就可以获得一个动态数字 token。我们来一起探索一下这门技术。

<!-- more -->

## TOTP

TOTP 即 [Time-Based One-Time Password Algorithm](https://tools.ietf.org/html/rfc6238)。顾名思义这是一种基于时间的 token 算法，从宏观来看就是基于时间和共享的密钥（shared secret key）生成随机数字 token。

与之类似的算法还有HOTP（[Event-Based One-Time Password Algorithm](http://tools.ietf.org/html/rfc4226)）。

## Google Authenticator

Google Authenticator 也是一种 TOTP，只不过它的 secret key 必须使用 base32 的[^1]。

## Rust Implement

* [Google Authenticator Rust](https://crates.io/crates/google-authenticator)：一个非常简介易用的 crate，但是示例都存在着一些问题，可用示例可以参考[我的 fork](https://github.com/jtr109/google-authenticator-rust)[^2]；
* [rust-oath](https://crates.io/crates/oath)：使用数最多的相关 crate，功能全面；

## Have A Try

* 在[项目](https://github.com/jtr109/rust-play/tree/otp/src/bin)中尝试了生成 secret （`secret.rs`），生成二维码（`qr_code.rs`）以及生成 token（`code.rs`）。

## Reference

* [^1]: Google Authenticator 的特殊性参考[这个项目中的说明](https://github.com/avacariu/rust-oath#google-authenticator)

* [^2]: [已提交 pull request](https://github.com/hanskorg/google-authenticator-rust/pull/3)