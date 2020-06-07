---
title: "在 Typora 和 Hugo 中正确显示图片"
date: 2020-06-07T19:48:37+08:00
draft: true
typora-root-url: ../../static/images
---

## 背景

Typora 可以便捷地将文件保存在本地，但是如果不合理配置，无法适应静态网页生成器（如：Hugo）的静态文件存储方式。所以需要通过 Typora 强大的配置方式解决这个问题。

## 目标

将文件存放在项目目录中的 `static/images` 目录下，同时保证能在 Typora 和 Hugo 生成的网站中可见。

## 配置 Typora

Typora 有两个选项需要配置：

1. 图片读取的根路径。通过指定这个参数，可以使图片文件以绝对路径的方式在 Markdown 文件中显示，从而适配生成网站中图片的 URL。
2. 图片的自动存放路径。

### 指定图片根路径

点击 Use Image Root Path，选择 `static/images` 目录作为根路径。

![image-20200607195608916](/image-20200607195608916.png)

### 设置图片自动存放路径

在设置中选择 Image 栏，将 `static/images` 的绝对路径设置为图片插入地址。

![image-20200607195835475](/image-20200607195835475.png)

经过如上配置，截图便可通过 Command+v 自动保存在 `static/images` 目录中。

## 配置 Hugo

但是此时如果拉起网站，图片不可见，原因是 Hugo 默认读取的路径为 `static`，所以我们需要在 `config.toml` 文件中修改配置：

```toml
staticDir = ["static/images"]
```

此时拉起站点，就可以正常显示图片了！

## 参考

* [使用hugo搭建博客系统](https://blog.dianduidian.com/post/使用hugo搭建博客系统/)
* [Images in Typora](https://support.typora.io/Images/)