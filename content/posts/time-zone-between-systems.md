---
title: "开发过程中的时间和时区问题"
date: 2020-06-09T12:03:19+08:00
typora-root-url: ../../static
draft: false
tags:
  - Datetime
  - Time Zone
  - Time
---

## 背景

Time zone 一直是一个老生常谈的问题，不同地区会划分出不同的时区（CST），同一个地区在不同季节也可能有不同的时间规则（DST）。在这里我们一起梳理一下开发过程中可能会遇到的时区问题。

## Naive Datetime & Timezone-awared Datetime

在 Python 中，我们会遇到两种显示时间的方式。转换到 ISO 8601 中，他们的显示也有差异：

* Naive dateime

  *  `2020-06-09T12:10:19.805`

* Timezone-awared datetime

  *  `2020-06-09T04:10:19.805Z`

  * `2020-06-09T04:10:19.805+00:00`

Naive datetime 是没有「时区」概念的，他表示的时间通常被用做「当地时间」，然而遇到跨时区的情况，两地无法区分这个时间指代的是什么时区，导致混乱。所以当你的服务或者用户处于不同时区，**绝对**不能使用 naive datetime。

而 Timezone-awared datetime 最后的 `Z` 或 `+00:00` 表明了时区，可以帮助不同时区的程序和系统以合适的方式显示时间。

## ISO Format & Timestamp

### ISO Format String 和 Timestamp 各自的特点

首先需要明确 ISO formated timezone-awared datetime和 timestamp 在服务间通讯能达到的效果是一样的，都可以无歧义地表达一个时间。让我们来分析一下他们的特点：

* ISO formated timezone-awared datetime
  * 字符串形式
  * 可读性强
  * 对时区表达方式不唯一（`Z` 或 `+00:00`）
* timestamp
  * 整型数字
  * 不同程序对 timestamp 精度的定义不同
    * 如 Java 和 JavaScript 中为 millisecond
    * 而 Unix 和 Python 中为 second

### 前端对时间的处理

如果浏览器当前时区为东八区，取一个北京时间 `2020-06-09T14:56:00.658540+08:00` 为例，对应的秒级精度 timestamp 为 1591685760.65854。

```js
> new Date("2020-06-09T14:56:00.658540+08:00")
< Tue Jun 09 2020 14:56:00 GMT+0800 (China Standard Time)  // 使用带时区 ISO format 可以正确解析

> new Date("2020-06-09T06:56:00.658540Z")
< Tue Jun 09 2020 14:56:00 GMT+0800 (China Standard Time)  // Z 结尾的 ISO format 也可以正确解析

> new Date("2020-06-09T14:56:00.658540")
< Tue Jun 09 2020 14:56:00 GMT+0800 (China Standard Time)  // 不带时区的 ISO format 会被当作浏览器当前时区解析

> new Date(1591685760658.54)
< Tue Jun 09 2020 14:56:00 GMT+0800 (China Standard Time)  // 毫秒级精度的时间戳可以正确解析

> new Date(1591685760.65854)
< Mon Jan 19 1970 18:08:05 GMT+0800 (China Standard Time)  // JavaScript 只能理解毫秒级的时间戳，秒级精度的时间戳会解析到错误时间
```

### 使用 ISO Format String 还是 Timestamp？

根据两种表示方案的特点，我们可以这样选择：

1. 在后端系统见传递时间，使用时间戳。另外考虑到提供更高的精度，推荐使用毫秒级时间戳。
2. 和前端的接口中使用带时区的 ISO Format String，提供更好的可读性，同时保证解析无歧义。

## 延伸阅读

* [Django 中自带的时区处理机制](https://docs.djangoproject.com/en/3.0/topics/i18n/timezones/)