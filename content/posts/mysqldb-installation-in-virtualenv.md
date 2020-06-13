---
title: Install MySQLdb in virtualenv，从入门到放弃
date: 2017-08-13T14:08:41+08:00
categories:
  - tech
tags:
  - Python
  - MySQL
  - Virtualenv
typora-root-url: ../../static
---

本人使用的是OSX，在安装MySQLdb的过程中遇到不少坑，和大家分享一下，也为以后留存。只是做一个整理，并不会有太多个人心得。多以评注为主。

# MySQL Server 密码重置

> 本机环境
> Mac OS X EI Capitan
> 官网 Community 版 MySQL Server
> 虚拟环境 virtualenv
> pwd: `/User/jtr109/qs/`

_密码重置的方式多种多样，用很多我尝试之后这个是我唯一可行的，和我有相同环境、情况的朋友可以参考：[数据库之mac上mysql root密码忘记或权限错误的解决办法](http://blog.csdn.net/u014410695/article/details/50630233)。_

遇到这个问题主要是我装MySQL的时候不专心，传说官方dmg装完后是会**提示设定密码**的，不知道怎么回事，反正我本地的MySQL Server就完全进不去了，只能改密码。这篇文章讲的非常清晰，简单易懂，不做过多评论。只要每一步照做就好。此文最大的优点是不用终端运行MySQL Server，也不用考虑 kill 进程，非常适合刚刚接触相关内容的朋友。

# 安装 MySQLdb

首先进入你需要的路径，我们以 `/User/jtr109/qs/` 为例。执行 `virtualenv venv` 创建虚拟环境包， `source venv/bin/activate` 进入虚拟环境。

## 注意安装模块名

我不知道为什么这个模块叫 MySQLdb，`import` 语句也是 `import MySQLdb`。但是你要**注意**，`pip` 安装语句为： `pip install MySQL-python`。千万不要写错。不过因为你在虚拟环境，所以玩脱了就这个环境包删了就好。这也是我为什么推崇为每个项目创建虚拟环境的原因之一。

## 修改 `_mysql.so` 追踪路径

现在你可以试着进入 python， 然后运行：

    >>> import MySQLdb
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/Users/jtr109/qs/venv/lib/python2.7/site-packages/MySQLdb/__init__.py", line 19, in <module>
        import _mysql
    ImportError: dlopen(/Users/qs/venv/lib/python2.7/site-packages/_mysql.so, 2): Library not loaded: /usr/local/opt/mysql/lib/libmysqlclient.20.dylib
      Referenced from: /Users/jtr109/qs/venv/lib/python2.7/site-packages/_mysql.so
      Reason: image not found

很好，你跟我遇到一样的问题了。

_此时你需要的是这篇文章：[Mac OS X下修复Reason: image not found](http://www.jianshu.com/p/e196d25a8e31)。当然，更重要的是里面给的另一个链接： [Python mysqldb: Library not loaded: libmysqlclient.18.dylib](http://stackoverflow.com/a/13421926)_

试着执行：

    $ otool -L /Users/jtr109/qs/venv/lib/python2.7/site-packages/_mysql.so
    /Users/jtr109/mystuff/qstrategy/venv/lib/python2.7/site-packages/_mysql.so:
        /usr/local/opt/mysql/lib/libmysqlclient.20.dylib (compatibility version 20.0.0, current version 20.0.0)
        /usr/local/opt/openssl/lib/libssl.1.0.0.dylib (compatibility version 1.0.0, current version 1.0.0)
        /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib (compatibility version 1.0.0, current version 1.0.0)
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)

_我的 `dylib` 是 `20` 的，请根据自身调整。_

来分析一下。第一个链接讲的比较清楚了，根据报错提示也可以看出，其实就是 `MySQLdb` 调用了 `_mysql` ，但是 `_mysql.so` 并没有在默认路径 `/usr/local/opt/mysql/lib/libmysqlclient.20.dylib` 找到这个文件。那接下来我们要做的就是改变他的搜索路径：

注意我们需要用到的 `install_name_tool` 语句的规则如下：

    install_name_tool [-change old new] input

几个参数的对应关系：

`dylib` 文件默认查找路径（`old`）: `usr/local/opt/mysql/lib/libmysqlclient.20.dylib`
我们需要 `dylib` 查找的路径（`new`）: `usr/local/mysql/lib/libmysqlclient.20.dylib`
修改的 `_mysql.so` 文件位置： `/Users/jtr109/qs/venv/lib/python2.7/site-packages/_mysql.so`

所以执行如下语句：

    $ sudo install_name_tool -change /usr/local/opt/mysql/lib/libmysqlclient.20.dylib /usr/local/mysql/lib/libmysqlclient.20.dylib /Users/jtr109/qs/venv/lib/python2.7/site-packages/_mysql.so

_这里有个很奇怪的点，两篇参考文章的 `old` 路径给的都只是文件名，我尝试下来并不能够改变 `_mysql.so` 中的查找路径。所以无论你是否按照我的语句改变查找路径，**请务必**再次执行 `otool` 确定 `_mysql.so` 中的查找路径是否更新了！_

确认更新后，再试试在python中运行 `import MySQLdb`。问题解决！

Tips: 这种解决方法意味着每次你新创建的 virtuanenv 中需要用到 MySQL，你都必须修改 `_mysql.so`。第一个链接中有一个自制的脚本用来执行这一操作，我没细看，大家可以参考一下。

那就写到这里吧，希望大家都能跑通。
