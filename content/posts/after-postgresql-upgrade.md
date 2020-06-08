---
title: Homebrew 自动更新 PostgreSQL 后对应数据库的更新处理
date: 2017-10-18T10:08:27+08:00
tags:
  - homebrew
  - postgresql
typora-root-url: ../../static
---

今天 PostgreSQL 更新到了 10.0, 对于我这样的新手, 可以说是悲喜交加. Server 本身更新了, 但是数据库还是老的, 会导致无法使用的问题. 在这里分享一下数据库的无痛恢复过程.

## 问题描述

今天执行 `$brew upgrade` 的时候, 发现 postgresql 也更新了, 心里生出一丝不详的预感. 打开本地开发的 web 服务:

```zsh
$/usr/local/bin/pg_ctl -D /usr/local/var/postgres restart
```

返回如下

```zsh
waiting for server to shut down.... done
server stopped
waiting for server to start..../bin/sh: /usr/local/Cellar/postgresql/9.6.5/bin/postgres: No such file or directory
/bin/sh: line 0: exec: /usr/local/Cellar/postgresql/0.6.5/bin/postgres: cannot execute: No such file or directory
stopped waiting
pg_ctl: could not start server
```

于是再运行

```zsh
$/usr/local/bin/pg_ctl -D /user/local/var/postgres start
```

想要启动数据库时, 就会发现出问题了:

```zsh
waiting for server to start... FATAL: database files are incompatible with server
[58146] DETAIL: The data directory was initialized by PostgreSQL version 9.6, which is not compatible with this version
10.0.
 stop waiting
pg_ctl: could not start server
Examine the log output.
```

如果你也遇到了这样的问题, 那就可以按接下来这样处理.

## 特别提示

- 由于我原来的版本是 9.6.5, 这里都以该版本举例, 其他版本的差异, 我会提及. 也请自己留意.
- 请找一个干净的目录执行以下操作, 因为执行成功之后, 会生成一些脚本和文件, 都是使用后可以删除的, 避免污染其他目录.

## 核心步骤

这个问题说白了就是数据库没有自动跟着 server 一起更新, 其实更新的时候 homebrew 下已经有对应的提示了.

我们主要要做的就是使用 `pg_upgrade` 将数据库版本更新, 具体的文档可以点击[这里](https://www.postgresql.org/docs/10/static/pgupgrade.html). 而对于我们而言, 核心的代码格式就是这样的:

```bash
$pg_upgrade \
-d <原数据目录> \  # 我们原来的数据库目录, 目前的版本为 9.6.5
-D <新数据目录> \  # 目前还不存在, 稍后会新建
-b <原执行目录> \  # 这个执行目录对我们而言已经不存在了
-B <新执行目录> \  # 更新后版本为 10.0 的执行目录
-v  # verbose internal logging, 最好写
```

## 实现依赖

从上面的格式可以看出, 我们需要 "新数据目录", "原执行目录", 那么开始补全

### 下载旧版本的 PostgreSQL

```bash
$brew tap caskroom/versions
$brew install postgresql@9.6  # 如果你的版本不是 9.6 请执行 $brew search postgresql, 根据返回的结果选择
```

### 初始化 10.0 数据库

```bash
$initdb /usr/local/var/postgres10.0 -E utf8
```

## 执行 `pg_upgrade`

```bash
$pg_upgrade \
-d /usr/local/var/postgres \
-D /usr/local/var/postgres10.0 \
-b /usr/local/Cellar/postgresql@9.6/9.6.5/bin \
-B /usr/local/Cellar/postgresql/10.0/bin \
-v
```

在一大堆的 ok 过后, 看到 `Upgrade Complete` 就说明安装成功了.

## 加载服务之前

这时候你可以做的事情有:

- 执行新生成的脚本, 如果你知道他们是干嘛的. (对不起, 我不知道. 欢迎交流! my email: `conbas2019@gmail.com`.)
- 删除老的执行目录和数据库目录. (建议别急着删除, 先观察一下, 我相信看到这里的同学都和我差不多水平, 那还请做事稳妥一点.)
- 用新路径替换老路径. (还是和上面一样的建议, 除非你知道自己在干嘛)

## 重新加载服务

由于我没急着删除各种文件, 先使用新的路径执行.

```bash
$/usr/local/bin/pg_ctl -D /usr/local/var/postgres10.0 start
```

如果显示

```bash
done
server started
```

那就说明成功了. 希望你也如此.

## 参考

- [Upgrading PostgreSQL 9.2 to 9.3 with Homebrew](https://mattbrictson.com/postgresql-93-brew-upgrade)
- [pg_upgrade](https://www.postgresql.org/docs/10/static/pgupgrade.html)
- [how to fix postgres after updating/upgrading brew](https://stackoverflow.com/questions/20754669/how-to-fix-postgres-after-updating-upgrading-brew) (`pg_dumpall` 也是一种方案, 不过我尝试失败了, 如果我的文章不能帮到你, 可以试试这个)
- [pgsql 10 的更新手册](https://www.postgresql.org/docs/10/static/upgrading.html) 提供了所有的更新方案


## 关于其他

自认为难得写了一篇有点用处的东西, 希望能帮到大家.

还是那句话, 希望多多交流. [这是我的 github](https://github.com/jtr109), 欢迎相互 follow.
