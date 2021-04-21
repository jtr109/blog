---
title: "HTTP 消息结束的标志"
date: 2021-04-21T16:46:17+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - network
  - HTTP
---

## 请求和响应

首先需要明确的一点是，无论是 HTTP 请求还是 HTTP 响应，都是由相同的 HTTP 消息构成[^message]。服务端判断 HTTP 请求结束和客户端判断 HTTP 响应结束的依据是一样的。

## HTTP 消息结构

首先让我们来了解一下 HTTP 消息的结构，从 RFC 2616 是这样描述[^construct]的：

> Both types of message consist of a start-line, zero or more header fields (also known as "headers"), an empty line (i.e., a line with nothing preceding the CRLF) indicating the end of the header fields, and possibly a message-body.

让我们看看更直观的语法定义：

```text
        generic-message = start-line
                          *(message-header CRLF)
                          CRLF
                          [ message-body ]
        start-line      = Request-Line | Status-Line
```

## 判断方法

接收方先将接收到的 HTTP 消息第一行作为起始行，在读到空行前的每一行作为消息头部。

### 一般情况

一般情况下，当读取完头部信息（读到两个连续的 CRLF[^crlf]），需要解析头部中的 Content-Length 消息头，根据消息头确认是否有消息体，以及消息体长度。之后即可读取指定长度的数据。当读取完消息体，意味着消息结束。

### Chunked

如果消息头中声明 `Transfer-Encoding: chunked`，那么意味着使用 [chunked transfer encoding](https://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.6.1)。我们先看一下  Chunked-Body 的定义：

```text
       Chunked-Body   = *chunk
                        last-chunk
                        trailer
                        CRLF
       chunk          = chunk-size [ chunk-extension ] CRLF
                        chunk-data CRLF
       chunk-size     = 1*HEX
       last-chunk     = 1*("0") [ chunk-extension ] CRLF
       chunk-extension= *( ";" chunk-ext-name [ "=" chunk-ext-val ] )
       chunk-ext-name = token
       chunk-ext-val  = token | quoted-string
       chunk-data     = chunk-size(OCTET)
       trailer        = *(entity-header CRLF)
```

可以看到 Chunked-Body 是 trailer 加上 CRLF[^crlf] 结尾。所以接收方应该不断读取消息体，直到读到两个连续的 CRLF[^crlf]，说明消息结束。

[^message]: RFC 2616 中是这样[描述](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html#sec4.1:~:text=HTTP%20messages%20consist%20of%20requests%20from%20client%20to%20server%20and%20responses%20from%20server%20to%20client.)的：HTTP messages consist of requests from client to server and responses from server to client.
[^construct]: 引用自 [HTTP/1.1: HTTP Message](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html)
[^crlf]: CRLF 的符号化表示是 `\r\n`，十六进制编码表示为 `0d0a`。

