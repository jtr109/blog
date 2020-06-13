---
title: flask 数据库迁移与部署的一些经验
date: 2017-08-13T13:04:28+08:00
categories:
  - tech
tags:
  - Flask
  - Python
typora-root-url: ../../static
---

最早发布于[flask开发的第五章关于创建迁移脚本的问题(已解决) - cocode.cc](http://cocode.cc/t/flask/6183/8?u=jtr_109)。内容并不完善，基本不会再维护.

分享一点经验，主要是自己的理解，希望能有所帮助。千辛万苦部署好的数据库，就不为了答题一一验证了，全凭记忆回答，有不对的地方欢迎指出，会及时修改。

以下内容以执行文件名为`manage.py`举例，请自行调整。

1. *影响 `migrate` 的是数据库的结构。*

    * 比方说你的 `User` 类中添加了一个 Column - `nickname` ，那么在保存 Model 之后建议执行 `python manage.py db migrate -m "add nickname`；
    * 而给 table 加入了新的方法，例如 `def change_nick_name()` ，是不需要 `migrate` 的。如果 `migrate` ，结果也是 `No change`；
    * 这一点可以留意一下，比较保险的办法是觉得有必要的时候就尝试一下 `migrate`，反正 `No changes` 对你的迁移也没有影响。

2. *在创建本地数据库的时候，不要在 `py shell` 环境下执行 `db.create_all()`*
    * 请使用`python manage.py db init`, `python manage.py db upgrade` 来创建本地数据库。
    * 通常我会模仿书本上在上面两条指令之间再做一次`migrate`，但经验告诉我，结果好像都是`No changes`，毕竟，我`init`之后没有做过调整啊。
    *  `init`指令会根据`Model.py`中的结构创建迁移文件夹`migrations`，而`upgrade`会根据`migrations`下的内容，通过`config.py`在找到正确的位置，并创建SQL，并在SQL中创建一个名为`versions`的table来存放和迁移文件夹中相同的版本号。（**重要**）
    * 在开始的尝试中，特别是往heroku上部署的过程中，经常会遇到许多数据库的坑，如果实在走投无路，又确定是数据库迁移问题，并且无法优雅地解决。你需要做的就是以下几步：
      (1). 休息一下，我知道你很累了
      (2). 执行`python manage.py db shell`，在`py shell`环境下执行`drop_all()`；或者直接删了你的原有数据库。
      (3). 按照本段前面的提示重新创建本地SQL
      (4). `drop_all()`或者删除并重新指定远端数据库
      (5). 执行`heroku run python manage.py db deploy`（假设你部署在heroku上，并已经完成了deploy的定义）
      (6). 现在你的远端和本地数据库的版本是相同的了。

3. *如果本地使用sqlite作为数据库，千万不要从`model.py`中删除`column`！*
    * 这一点非常重要，因为根据我查到到资料显示，sqlite是没有“删除行”这一个选项的。如果你删除了某个`column`，这一操作能被`migrate`正确记录，但是在你`upgrade`的时候就会报错。然后想要还原migrate版本？抱歉，我不会……（欢迎朋友们指导一下，谢谢！）

4. *部署到远端时，请确认你又`versions`文件夹*
    * 我不确定`init`会不会有，在执行`migrate`指令时，会更新`versions`文件夹
    * 如果没有`versions`文件夹，远端部署的时候也会报错，请务必留意。

5. *理解每个迁移指令的含义*
    * `init`的作用是初始化迁移，创建`migration`文件夹
    * `migrate`的作用是根据model.py中的变化更新迁移，该指令会在`migration/versions/`文件夹中新增一个版本文件，文件名为版本号。打开能够看到对应model.py文件，修改前后新迁移版本发生的变化。
    * `update`会将迁移中最新版本的结构应用到数据库中（例如添加新增的列），并更新自动创建的`version` 表中的版本号。该版本号与`migration/versions/`文件夹中对应。任何情况下，这两个版本号不对应都会导致出错。
