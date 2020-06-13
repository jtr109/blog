---
title: CSS 学习笔记
date: 2017-08-13T12:30:45+08:00
categories:
  - Tech
tags:
  - CSS
typora-root-url: ../../static
---

## 居中

### 1. 文字居中

- 水平居中: `text-align: center`
- 垂直居中: `vertical-align: middle`

### 2. div 居中

_div 居中是一个比较坑的实现, 没有什么特别优秀的办法, 这里只举部分较实用的例子._

具体方法可以参考[知乎上的这一篇回答](https://zhihu.com/question/20543196/answer/89218605), 这里暂时只介绍其中 absolute 的使用.

#### 2.1 absolute 居中

```css
// 假设我们要做一个 200px 见方的 div
.middle {
  position: absolute;
  width: 200px;
  height: 200px;
  top: 50%;
  left: 50%;
  margin-top: -100px; 
  margin-left: -100px;
  background: red;
}
```

这是一个比较稳妥的实现, 这里很明显, 是通过 `top: 50%` 来实现的垂直居中, 不过如果单单使用 `top=50%`, 会发现 div 的顶部位于父级元素 50% 位置处, 所以需要使用 `margin-top=-100px` 来实现元素的便宜. 该示例中的水平居中方法同理.

如果在确认设置无误的情况下, 发现 div 的位置不对, 请检查父级是否正确设置了 `position: relative`.

## SCSS

SCSS 的 [variable](https://sass-lang.com/guide#topic-2) 和 [mixin](https://sass-lang.com/guide#topic-6) 在生产的应用比较广泛, 具体使用方法可以直接参考[官方文档](https://sass-lang.com/guide).

## 文字省略

文字省略的方法可谓千奇百怪, 这里介绍两种常用的方式, 分别对应单行省略和多行省略.

### 单行省略

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

### 多行省略

以2行省略为例:

```css
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 2;
-webkit-box-orient: vertical;
```

### 结合 SCSS mixins 使用

多行省略需要使用的场景比较广泛, 然而每种行数都写一个对应的类, 这样也不够优雅. 比较推荐的做法就是把它和 SCSS mixins 结合起来:

```css
@mixin multiple-lines ($count) {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: $count;
  -webkit-box-orient: vertical;
}

.two-lines {
  @include multiple-lines(2);
}
```
