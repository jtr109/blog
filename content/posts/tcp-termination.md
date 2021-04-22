---
title: "TCP 挥手动作大赏"
date: 2021-04-22T15:43:41+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - TCP
---

## 四次挥手还是三次挥手

我们通常说的四次挥手是指：

1. 客户端完成数据传输后，发送 FIN
2. 服务端接收到客户端的 FIN 后发送 ACK 确认收到
3. 服务端完成消息发送后，发送 FIN
4. 客户端接收到服务端端 FIN 后发送 ACK 确认收到

但是实践中也有很多情况下服务端会把 2、3 阶段的 ACK 和 FIN 合并发送，从而通过三次消息发送实现了挥手[^three]。

## 形态各异的挥手

标准是标准，但是各个网站对挥手的处理方式可谓五花八门，让我们来见识见识不同的网站是怎么处理的。

### 微信

![image-20210422155103336](/images/tcp-termination.assets/image-20210422155103336.png)

微信的服务端在挥手时（No. 17995）没有把 Ack 加 1，导致客户端混乱。客户端只能理解为服务端中断，发送了一个正常的分组（No. 17997）来结束 TCP 连接。

样本来自我 Mac 上微信客户端的一个 HTTP 请求，我也不知道是干嘛的。

<img src="/images/tcp-termination.assets/image-20210422162514104.png" alt="image-20210422162514104" style="zoom:50%;" />

### 百度

通过下面的命令请求百度首页。

```bash
curl http://www.baidu.com
```

![image-20210422155732834](/images/tcp-termination.assets/image-20210422155732834.png)

可以看到百度的挥手协议非常标准，可喜可贺。

### GitHub

通过下面的命令请求 GitHub 首页。

```bash
curl http://www.github.com
```

![image-20210422155854529](/images/tcp-termination.assets/image-20210422155854529.png)

可以看到 GitHub 的服务端将 FIN 和 ACK 合并在了一起，只发送了一个请求。所以挥手只进行了三步。

### 知乎

通过下面的命令请求知乎首页。

```bash
curl -v http://www.zhihu.com
```

![image-20210422160446638](/images/tcp-termination.assets/image-20210422160446638.png)

可以看到知乎也只进行三次挥手。

### 新浪

通过以下命令请求新浪首页。

```bash
curl -v http://www.sina.com
```

![image-20210422160942682](/images/tcp-termination.assets/image-20210422160942682.png)

新浪也只做了三次握手。

[^three]: 参考 Wikipedia 的[说明](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#Connection_termination:~:text=It%20is%20also%20possible%20to%20terminate,host%20A%20replies%20with%20an%20ACK.%5B17%5D)。