---
title: "使用 macOS Wireless Diagnostics 选择合适的 Wi-Fi 信道"
date: 2020-09-15T23:19:50+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Wi-Fi
---

## Wi-Fi 信道

我们在设置路由器的时候通常会看到「信道选择」的参数。

查阅资料可以很容易了解到，同一个信道中如果有多个 Wi-Fi 设备，会造成相互干扰。为了避免这样的状况，我们应该为 Wi-Fi 路由器选择一个合适的信道来避免干扰。可是如何选择合适的信道呢？

其实 macOS 中已经自带了一个应用 Wireless Diagnostics，通过它可以很方便地了解到当前环境中的 Wi-Fi 信道情况。

## 使用 Wireless Diagnostics 选择信道

### 打开应用

首先我们需要打开 Wireless Diagnostics。可以启用 Alfred 或者 Spotlight，输入 Wireless Diagnostics 即可打开。或者按住 <kbd>option</kbd> 键，点击右上角的 Wi-Fi 图标，在选单中选择 "Open Wireless Diagnostics..."。

![image-20200915232936215](/images/using-mac-wireless-diagnostics-to-choose-wifi-channels.assets/image-20200915232936215.png)

### 打开 Scan 窗口

打开 Wireless Diagnostics 默认弹出的窗口与我们无关，忽略即可。点击菜单栏中的 Window -> Scan，打开 Scan 窗口。这时窗口的左侧显示的就是 Wi-Fi 信道的最有选项。

进入路由器设置页面，根据推荐的信道设置即可。