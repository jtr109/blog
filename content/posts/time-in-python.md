---
title: "Python 中的事件处理"
date: 2020-10-06T11:16:30+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Python
  - time
---

## 时间差计算

计算时间差最直观的方式是使用 `time.time` 方法：

```python
import time

t1 = time.time()
# Processing
t2 = time.time()

duration = t2 - t1
```

然而 `time.time` 是基于墙上时钟（或称钟表时间），该事件通过进程所在的操作系统和远端 NTP 服务器更新。这意味着两次 `time.time` 之间可能发生过始终同步，导致记录的时间间隔不可靠，甚至可能出现负值。

在 Python3 中，更合适的做法是通过 `time.monotonic` 调用系统的单调时钟：

```python
import time

t1 = time.monotonic()
# Processing
t2 = time.monotonic()

duration = t2 - t1
```

由于单调时钟是由单机维护的，它总是向前，不会受始终同步影响，所以其记载的事件更为准确。