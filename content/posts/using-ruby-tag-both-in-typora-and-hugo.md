---
title: "在博客或文档中使用 Ruby Tag 实现注解效果"
date: 2020-06-13T12:02:11+08:00
typora-root-url: ../../static
draft: true
tags:
  - HTML
---

## 背景

在博客或文档中，我们有时候会希望对自己的文字做注解。例如为一个专业术语加上标注，便于读者理解：

> We are about to study the idea of a *computational process*.

我们会翻译为：

> 我们将要学习计算过程的概念。

如果我们想要引述原文来增加准确性，一个简单的做法是在后面加上括号：

> 我们将要学习计算过程 (computational process) 的概念。

有没有更好的方法呢？

## 使用 HTML 元素 Ruby Tag

在 HTML 中，我们可以使用 Ruby Tag 来实现这个需求。

### Ruby Tag 概述

这里简单解释一下 ruby tag 的整体结构：

```html
<ruby>
  计算过程
  <rp>（</rp>
  <rt>computational process</rt>
  <rp>）</rp>
</ruby>
```

`<ruby>` 中的元素主要分为几部分：

* 主体，即示例中的「计算过程」。这部分是正常显示在段落中的内容，其格式继承自 `<ruby>` 外部的格式。
* `<rp>` 用来包裹括号。这是为了兼容性，在不支持 `<ruby>` 的浏览器中显示为「计算过程（computational process）」
* `<rt>` 用来包裹注释信息，注释会以小字形式显示在主体内容的上方。

### 实践示例

```
我们将要学习<ruby>计算过程<rp>（</rp><rt>computational process</rt><rp>）</rp></ruby>的概念。
```

实现的效果如下：

> 我们将要学习<ruby>计算过程<rp>（</rp><rt>computational process</rt><rp>）</rp></ruby>的概念。

## 参考

* https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ruby