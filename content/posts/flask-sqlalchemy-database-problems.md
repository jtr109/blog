---
title: 关于flask-sqlalchemy中数据库操作的问题整理
date: 2017-08-13T14:06:44+08:00
tags:
  - flask
  - python
typora-root-url: ../../static
---

此文为本人原创，最早发布于[求助《Flask Web 开发》第五章关于数据库db.session.add()和db.session.commit()问题](http://cocode.cc/t/flask-web-db-session-add-db-session-commit/6867)。


问题简述：

> 当加入了`SQLALCHEMY_COMMIT_ON_TEARDOWN=True`，在什么情况下仍然要执行`db.session.commit()`

解答整理如下：

`config`是在`app/__init__.py`中调用的，所以执行`model.py`中的方法时，`SQLALCHEMY_COMMIT_ON_TEARDOWN = True`已经生效了。即请求结束后自动commit。

后面几个章节中出现`db.session.commit()`的主要作用都是为了生成id来使用。

你可以结合源码看一下。

session的原理就请参考[Flask 源码阅读笔记](http://blog.csdn.net/yueguanghaidao/article/details/40016235)。


`SQLALCHEMY_ON_TEARDOWN`在flask-sqlalchemy 2.0之后已经被删除了，需要用`SQLALCHEMY_TRACK_MODIFICATIONS`替代，想必你运行的时候应该看到过相关提示。

参考文章：[Flask-SQLAlchemy 配置问题](http://www.jianshu.com/p/70c892fab7af)

这个新熟悉我还没掌握，抽空看下documentation再来分析。

继续回到`SQLALCHEMY_ON_TEARDOWN`。按照书上的说法：

> 将其设为True时，每次请求结束后都会自动提交数据库中的变动。

因为v1.0的如果这里的documentation已经找不到了，没有依据，我就在这推测一下：

只有当请求结束的时候，才会自动`commit`，而直接操作数据库是不行的，需要手动`commit`。

回到model.py中看，`insert_roles()`被`@staticmethod`包装，是一个静态方法，它执行的位置是在`python manage.py shell`中，而非请求中，所以需要手动`commit`。

你可以验证一下，方法也很简单，保证写入`SQLALCHEMY_ON_TEARDONW=True`后，在shell中尝试：

    >>> user = User(username='test')
    >>> db.session.add(user)
    >>> quit()

再次进入shell，查找`<User u'test'>`，不存在，即推断合理：shell中执行的用于操作数据库的方法还是要加`commit`。
