---
title: "字节计量单位探究"
date: 2020-07-02T11:34:13+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - byte
  - network
---

在计算机使用过程中，我们会遇到困惑，有的时候我们使用 KB、GB 这样的十进制表示（如磁盘空间），有的时候我们又可以看到 KiB，GiB 这样的表示方式（如网络传输和 Kubernetes 配置）。

## 区别

字节的表示方法分为两类，如 KB、MB、GB 这类为十进制表示法，而 KiB、MiB、GiB 这类则是二进制表示法。[^1]

![Wiki 中提供的对比图表](/images/byte.assets/image-20200702114319210.png)

由于历史原因，存储设备使用国际单位制度（十进制前缀），网络传输使用 IEC 60027-2（二进制前缀）。

## 特殊情况

Windows 和 macOS 以及硬件厂商在计量磁盘空间时选择的单位也分为两个阵营[^2]。

> Gibibyte与[Gigabyte](https://zh.wikipedia.org/wiki/Gigabyte)常常被混淆，前者的计算方式是二进制，后者的计算方式是十进制。现今的计算上，常把Gigabyte以二进制的方式计算，即230。(因为Windows对*GB*这个信息计量单位的误用，因此在Windows中显示的"1GB"，其实应是指"1GiB"，由此常造成误解。并且因Windows操作系统的占有率高，导致该误用非常普遍。)，由于两种换算方法的不同，使容量在计算上相差了7.3%，所以常有Windows系统报告的容量比硬盘标示的容量还要小的情况发生。但在[苹果公司](https://zh.wikipedia.org/wiki/苹果公司)的[OS X](https://zh.wikipedia.org/wiki/OS_X)操作系统中，对于[存储设备](https://zh.wikipedia.org/wiki/儲存裝置)的容量计算方式与硬件厂商一致，均为1GB = 1,000,000,000（109）字节的十进制，避免了计算和使用上的麻烦。

## 参考

* [Why does Explorer use the term KB instead of KiB?](https://devblogs.microsoft.com/oldnewthing/20090611-00/?p=17933)

[^1]: 截图来自[维基百科「字节」词条](https://zh.wikipedia.org/wiki/字节)
[^2]: 摘自[维基百科 Gibibyte 词条](https://zh.wikipedia.org/wiki/Gibibyte)

