---
title: TCP 滑动窗口
date: 2021-04-21T14:08:01+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - network
  - TCP
---

## 背景

TCP 滑动窗口协议是为了帮助发送端了解数据处理的带宽或者数据处理速度，避免拥塞，从而解决包乱序问题，提供可靠传输[^target]。

## 工作机制

### 整理概念

TCP 协议中，当接收方接收到发送方发送的数据时，会返回一个 ACK 确认信息。ACK 中包含了两个重要信息：

* 序号（sequence number）

* 窗口大小（window）

如果接收方已经接收了前 $n-1$ 个字节，ACK 中返回的序号为 $n$，代表希望接收到从第 $n$ 个字节开始的数据。发送方应该根据序号确认接收方已经接收到了前 $n-1$ 个字节，而不用重复发送。

发送 ACK 前，接收方根据自己的缓存区空间计算还可以接受多少字节数据，记录在 ACK 的 window 中，我们假设值为 $m$。发送方收到确认消息后，知道自己最多再发送 $m$ 个字节的数据。

但是虽然此时发送方收到了接收方确认的前 $n-1$ 个字节，但是在此之前发送方可能已经发送到了序号 $p$。这时候发送方为了避免发生拥塞，最多只能发送 $m - (p - n)$ 个字节的数据。[^zhihu]

![TCP 头的格式](https://nmap.org/book/images/hdr/MJB-TCP-Header-800x564.png)

图片来源：[TCP Header | Nmap Network Scanning](https://nmap.org/book/tcpip-ref.html#tcp-header)

### 从发送方视角理解滑动窗口

![](http://www.tcpipguide.com/free/diagrams/tcpswwindows.png)

图片来源：[The TCP/IP Guide - TCP Sliding Window Acknowledgment System For Data Transport, Reliability and Flow Control](http://www.tcpipguide.com/free/t_TCPSlidingWindowAcknowledgmentSystemForDataTranspo-6.htm#Figure_207)

发送方发送完一部分数据后，不会等待接收方的确认消息，而是在继续发送其余的数据。所以在某个时间节点，对发送方而言存在 4 类数据：

1. 已经确认接收成功的数据
2. 已经发送但接收方还没有确认接收到的数据
3. 还未发送的处于窗口范围内的数据
4. 还未发送的处于窗口范围外的数据

上面的 2、3 部分构成了窗口数据，发送方根据接收方返回的 ACK 中的 wIndow 计算第 3 部分的数据量，并继续发送。

需要注意的是，这个窗口的位置并不是静止的，当发送方收到了新的 ACK 响应时，会根据 ACK 中的信息进行窗口滑动。

![](http://www.tcpipguide.com/free/diagrams/tcpswslide.png)

图片来源：[The TCP/IP Guide - TCP Sliding Window Acknowledgment System For Data Transport, Reliability and Flow Control](http://www.tcpipguide.com/free/t_TCPSlidingWindowAcknowledgmentSystemForDataTranspo-8.htm#Figure_209)

## 延伸阅读

以上只是对 TCP 滑动窗口的粗略介绍，进一步理解可以参考：

* 酷壳的两篇 TCP 文章：
  * [TCP 的那些事儿（上） | 酷 壳 - CoolShell](https://coolshell.cn/articles/11564.html)
  * [TCP 的那些事儿（下） | 酷 壳 - CoolShell](https://coolshell.cn/articles/11609.html#TCP%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3)
* [The TCP/IP Guide - TCP Sliding Window Acknowledgment System For Data Transport, Reliability and Flow Control](http://www.tcpipguide.com/free/t_TCPSlidingWindowAcknowledgmentSystemForDataTranspo.htm)

[^target]: 参考[酷壳](https://coolshell.cn/)中对[滑动窗口](https://coolshell.cn/articles/11609.html#TCP%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3)的介绍。
[^zhihu]: 参考[知乎回答](https://www.zhihu.com/question/32255109/answer/68558623)