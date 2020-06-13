---
title: "VSCode 字体配置"
date: 2020-06-13T20:44:59+08:00
typora-root-url: ../../static
draft: false
categories:
  - tech
tags:
  - VSCode
  - font
  - editor
---

## 背景

在修改 VSCode 的 color theme 时会看到一些 theme 的效果图中使用了非常好看的手写体字样。

![https://raw.githubusercontent.com/ahmadawais/shades-of-purple-vscode/master/images/JavaScript.png](https://raw.githubusercontent.com/ahmadawais/shades-of-purple-vscode/master/images/JavaScript.png)

图片来自 [Shades of Purple](https://marketplace.visualstudio.com/items?itemName=ahmadawais.shades-of-purple)

## 实践

### 字体选择

在 Shades of Purple 提供的[推荐配置](https://marketplace.visualstudio.com/items?itemName=ahmadawais.shades-of-purple#best-custom-settings)中可以了解到，截图中使用的字体是 [Operator Mono](https://www.typography.com/fonts/operator/overview)。可以在 [CUFON Fonts](https://www.cufonfonts.com/font/operator-mono) 网站免费下载[^1]。

### 安装字体

下载并解压字体包，可以看到各种 Operator Mono 字体。逐一双击安装。

![VSCode%202a432b75b011476185f70e223ceaaf31/Untitled.png](/images/vscode-font-configuration.assets/Untitled.png)

![VSCode%202a432b75b011476185f70e223ceaaf31/Untitled%201.png](/images/vscode-font-configuration.assets/1.png)

![VSCode%202a432b75b011476185f70e223ceaaf31/Untitled%202.png](/images/vscode-font-configuration.assets/2.png)

### 配置 VSCode 使用的字体

打开 VSCode，打开设置，搜索 *font family*，将 Operator Mono 设置在最前。

![VSCode%202a432b75b011476185f70e223ceaaf31/Untitled%203.png](/images/vscode-font-configuration.assets/3.png)

### 下载和使用合适的 Theme

如果你当前的 color theme 已经有较好的语法高亮规则，你就可以看到自己的配置已经生效，字体变为 Operator Mono 了。

如果不知道用什么 theme 比较好，可以考虑尝试 [Night Owl](https://marketplace.visualstudio.com/items?itemName=sdras.night-owl) 或者 [Shade of Purple](https://marketplace.visualstudio.com/items?itemName=ahmadawais.shades-of-purple)。

![VSCode%202a432b75b011476185f70e223ceaaf31/Untitled%204.png](/images/vscode-font-configuration.assets/4.png)

图片中的 theme 为 Shade of Purple，可以看到注释中的字体变成了斜体。

### 中文字体

大部分的流行字体都是 ASCII 类型，没有提供中文字体支持。所以 VSCode 中的中文字体会显示默认字体。如果有喜欢的中文字体，可以写在 Operator Mono 后面。VSCode 会按序查找字符集。

我使用的是方正和可口可乐推出的[在乎体](https://www.foundertype.com/index.php/FontInfo/index/id/4792)。个人可免费使用。感兴趣的朋友可以尝试一下。

![VSCode%202a432b75b011476185f70e223ceaaf31/Untitled%205.png](/images/vscode-font-configuration.assets/5.png)

## 补充说明

### 字体差异

在这里简单说明一下几种字体的差异：

文字粗细从粗到细：

- Bold
- Medium
- Book
- Light
- XLight

### Italic 后缀字体

*Italic* 后缀说明了这个字体是否是纯斜体，而非 italic 字体提示同时包含了正体和斜体字形。

![VSCode%202a432b75b011476185f70e223ceaaf31/Untitled%206.png](/images/vscode-font-configuration.assets/6.png)

如果直接使用 italic 后缀的字体，你的界面会变成全斜体。

[^1]: 官方字体需要购买，不清楚为何 CUFON Fonts 可以直接免费下载。下载是否侵权，请各位自行判断。懂行的朋友可以评论指导一下，如果涉嫌侵权，可联系我删除链接。