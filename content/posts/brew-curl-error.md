---
title: Homebrew 报错汇总
date: 2017-09-06T11:50:17+08:00
tags:
  - homebrew
typora-root-url: ../../static
---

## curl 超限错误

```
curl: (18) transfer closed with ** bytes remaining to read
```

这个报错是由于 mac 自带的 curl 请求头中长度的限制导致的, 我们需要按照由 homebrew 管理的 curl 和 git 来避免这个问题:

```bash
$ brew install curl --with-openssl
$ brew install git --with-curl --with-openssl
```

暂时解决方法不详
