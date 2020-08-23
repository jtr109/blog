---
title: "使用 Telnet 模拟邮件客户端发送邮件"
date: 2020-08-22T23:10:13+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - SMTP
---

## 基础知识

SMTP 协议中有3个组成部分：

- 用户代理（user agent）
- 邮件服务器（mail server）
- 简单邮件传输协议（Simple Mail Transfer Protocol, SMTP）

邮件服务器分为两类：

* 客户端邮件服务器
* 服务端邮件服务器

一个邮件的声明周期为：

```
发件人用户代理 -(1)-> 客户端邮件服务器 -(2)-> 服务端邮件服务器 <-(3)- 收件人用户代理
```

其中 (1) (2) 通过 SMTP 协议实现，(3) 为 POP3 或 IMAP 协议。

本文将使用 telnet 模拟用户代理，遵循 SMTP 协议向客户端 SMTP 服务器验证用户身份并发送请求（即 (1) 阶段），实现发送邮件动作。

## 操作步骤

### 数据准备

首先我们需要准备好用户名和密码，由于 SMTP 协议中登录用的用户名和密码都必须是 base64[^1] 的。

```shell
$ echo -n "sender@example.com" | base64
c2VuZGVyQGV4YW1wbGUuY29t
$ echo -n "yourpassword" | base64
eW91cnBhc3N3b3Jk
```

### 建立 TCP 连接

以下以 `smtp.example.com` 作为客户端邮件服务器举例。

首先通过 telnet 和客户服务器 SMTP 服务器建立连接。

```shell
$ telnet smtp.example.com 25
Trying 220.181.97.88...
Connected to smtp.example.com.
Escape character is '^]'.
220 telproxy-sm3.hmbj.internal ESMTP ready
```

首先我们建立了一个 TCP 连接，该连接保持在连接状态。我们向服务器发送 `HELO` 命令，服务器返回响应提供服务器可以处理的命令：

```
EHLO smtp.example.com
250-telproxy-sm3.hmbj.internal
250-PIPELINING
250-8BITMIME
250-AUTH=LOGIN PLAIN
250-AUTH PLAIN LOGIN
250 STARTTLS
```

### 验证身份

我们使用 `AUTH` 验证用户身份，指定使用 `LOGIN` 协议。服务器返回响应状态码和信息。信息经过了 base64 编码，解码可以看到要求填写的信息分别为 `Username:` 和 `Password:`。

```
AUTH LOGIN
334 VXNlcm5hbWU6
c2VuZGVyQGV4YW1wbGUuY29t
334 UGFzc3dvcmQ6
eW91cnBhc3N3b3Jk
235 2.0.0 OK
```

### 发送邮件

下面可以开始编写邮件内容：

```
MAIL FROM: <sender@example.com>
250 2.1.0 Ok
RCPT TO: <recept@example2.com>
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
From: sender@example.com
To: recept@example2.com
Subject: Test Sending Email with Telnet

Hello!
.
250 2.0.0 Ok: queued as 749D640F5E
```

这样邮件就一个成功发送了，最后让我们断开 TCP 连接：

```
QUIT
221 2.0.0 Bye
```

这样就完整模拟了一次用户代理向 SMTP 客户服务器发送邮件的动作。

[^1]: [协议标准](https://www.ietf.org/archive/id/draft-murchison-sasl-login-00.txt)供参考。