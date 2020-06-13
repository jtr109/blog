---
title: 计算机相关时间规则和表示方法
categories:
  - tech
date: 2019-12-07 22:02:45
tags:
  - Datetime
typora-root-url: ../../static
---

## ISO 指的是什么?

关于时间, 我们说的 ISO 一般是指 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)

ISO 8601 主要定义了时区0点的规则, 消除各种时间的表达方法. 我们可以在 [wiki 页面](https://en.wikipedia.org/wiki/ISO_8601)查看到当前时间的表达法示例:

| Name                  | Display                                                      |
| --------------------- | ------------------------------------------------------------ |
| Date                  | 2019-12-08                                                   |
| Date and time in UTC  | 2019-12-08T01:24:12+00:00<br/>2019-12-08T01:24:12Z<br/>20191208T012412Z |
| Week                  | 2019-W49                                                     |
| Date with week number | 2019-W49-7                                                   |
| Date without year     | --12-08                                                      |
| Ordinal date          | 2019-342                                                     |

### 标准时区

根据 Wiki 上的描述:

> In general, ISO 8601 applies to representations and formats of dates in the Gregorian (and potentially proleptic Gregorian) calendar, of times based on the 24-hour timekeeping system (with optional UTC offset), of time intervals, and combinations thereof.

可以看到 ISO 8601 采用格林威治时间作为标准时区.

### 时间表达

> Representations must be written in a combination of Arabic numerals and certain characters (such as "-", ":", "T", "W", and "Z") that are given specific meanings within the standard; the implication is that some commonplace ways of writing parts of dates, such as "January" or "Thursday", are not allowed in interchange representations.

计算机中对 ISO 8601 最常见的表达规则是 [RFC 3339](https://www.ietf.org/rfc/rfc3339.txt).

标准写法为:

```
2019-12-08T01:24:12.411112355+00:00
```

在 ISO 8601 中, 是不允许 `January`, `Thursday` 这样的表示方法的, 所有时间量都是用阿拉伯数字表示和特殊含义字符表示. 我们简单分析一下几种符号:

- `-`: 日期分隔符
- `:`: 时间分隔符
- `T`: 日期和时间之间的分隔符
- `W`: 周表示符号, 如 `W49` 表示第49周
- `Z`: 零时区表示, 即 `<time>Z` 和 `<time>+00:00` 含义相同.

## RFC 2822

标准的 ISO 8601 的月份和星期表达方式都是数字, 在英语语境下的使用场景看起来不是那么直观, 所以如果需要在时间字符串中使用 `Sun`, `Dec` 这样的表达方式, 需要使用 [RFC 2822](https://www.ietf.org/rfc/rfc2822.txt).

使用 RFC 2822 表示 ISO 8601 时间 `2019-12-08T01:24:12+00:00` 应当显示为:

```
Sun, 08 Dec 2019 01:24:12 +0000
```
