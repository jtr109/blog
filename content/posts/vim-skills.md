---
title: vim 使用技巧
date: 2017-08-13T11:50:29+08:00
categories:
  - Tech
tags:
  - Vim
typora-root-url: ../../static
---

## 删除行末的空格
```vim
:%s/\s\+$//e
```

有几点值得注意:

- `e` tag 的作用是 error no display, 这里加入的目的是为了在没有可替换内容的时候不发出错误提示.
- 可以发现这里的正则表达式中, "多个空格" 的处理是 `\s\+`, 这里需要留意, `+` 也是需要转义的.
