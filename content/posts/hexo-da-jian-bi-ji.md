---
title: Hexo 搭建笔记
date: 2017-08-11T07:41:21+08:00
categories:
  - Tech
series:
  - Blog Experimental Reports
tags:
  - Hexo
  - Blog
typora-root-url: ../../static
---

## 项目搭建

项目搭建可以参考这篇 limedroid 的[Hexo 博客搭建指南](https://github.com/limedroid/HexoLearning).

提供一条 bash one line:

```bash
$ hexo clean && hexo g && hexo s
```

上面的代码可以快捷地拉起项目服务器.

## 语言设置

如果你和我一样按照上面的文章配置了 Hexo, 你会发现自己的时间文字显示为德语 "Veröffentlicht am".

这是由于默认配置的语言不符. 首先确认 `theme/next/languages/` 目录下有 `zh-Hans.yml` 文件, 另外我们需要将 `_config.yml` 中配置语言:

```
language: zh-Hans
```

_需要注意, 修改好配置文件后请使用 `hexo clean && hexo g && hexo s` 重新编译后再查看效果._

参考:

- [使用Hexo踩坑小记](http://supermaryy.com/2016/07/02/Obstacles_I_Met_When_Using_Hexo/) (文章中还有其他使用技巧, 值得学习.)

## 关于文章更新

通常如果只是修改文章内容, 可以保持 hexo server 出于运行状态, 修改的内容会自动更新到我们本地的 `localhost:4000` 页面上. 不过在提交文章之前, 请务必再使用 `hexo g` 重新编译.

同样提供一条代码:

```bash
$ hexo g && hexo d
```

## 编译报错

在执行 `hexo clean && hexo g && hexo s` 时遇到过这样一个问题:

```bash
Error: Module version mismatch. Excepted 48, got 51.
...
```

具体原因不详, 有说法是由于 node 更新导致版本不一致导致的. 下面提供几种处理方式, 或许能解决问题.

1. `rm-rf node_modules && npm install --no-optional`
2. `npm rebuild hexo`
3. `npm uninstall hexo-cli -g && npm install hexo-cli  -g`

### 参考

- 基本所有解决方式都可以在[官方 issue](https://github.com/hexojs/hexo/issues/1939) 中找到解答.

## 页面请求极慢情况

使用 `hexo s` 在本地跑项目的时候, 发现主页加载极慢, 即使加载成功, 进入新的页面也加载极慢.

按理说本地请求静态文件是不可能有如此大的延迟的. 查看请求可以发现有一个对 fonts.googleapis.com 的请求, 经过了解, 发现这个请求是我们的 next theme 去请求的字体.

由于众所周知的原因, 这个网址是不存在的. 所以我们需要指定正确的 font resource. 这里我选择了[中科大的镜像](https://lug.ustc.edu.cn/wiki/lug/services/googlefonts#使用) 经过尝试, 可以秒开主页.
