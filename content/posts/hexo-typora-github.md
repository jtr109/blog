---
title: Post Blog with Hexo with Next
date: 2019-11-07 19:48:56
categories:
  - tech
series:
  - blog Experimental Reports
tags:
  - Hexo
  - Typora
  - Github
typora-root-url: ../../static
---

## Install Hexo Asset Image

```shell
npm install https://github.com/CodeFalling/hexo-asset-image -save
```

## Configure Hexo

编辑 `_config.yml`, 设置 `titlecase`, 该功能使你在 `hexo new "My New Post"`  的时候创建的文件自动带上 `-`, 形如 `My-New-Post`:

```yml
titlecase: true
```

## Create Post with Hexo

打开 `_config.yml` , 修改配置为 `post_asset_folder: true`

使用 Hexo 创建 post:

```shell
hexo new hexo-typora-github
```

这时 `source/_posts` 目录下会生成一个 `hexo-typora-github.md` 文件和一个 `hexo-typora-github` 目录, 这个目录就是图片的存放地址.

需要注意的是, 这里需要使用 `hexo-typora-github` 这样的格式生成. 如果使用 `hexo new "Hexo Typora Github"`, 生成的图片目录名为 `Hexo Typora Github`, 无法和文件映射.

## Editing with Typora

Typora 可以直接将图片复制到 `.md` 文档中, 但是如何和 Hexo 拉起服务的图片路径保持一致是一个问题.

使用 Typora 打开 `source/_posts` 目录, 点击设置 -> 图片:

1. 修改图片存放目录为 `./${filename}`
2. 勾选 `Use relative path if possible`.

![image-20191107200154200](/images/hexo-typora-github.assets/image-20191107200154200.png)

尝试截图, 使用 `Ctrl-v` 粘贴到 Typora 中, 图片地址为: `![image-20191107200154200](hexo-typora-github/image-20191107200154200.png)`. 符合预期, 图片可以正常显示了.

## Deploying to Github

修改 `_config.yml` 中的 `deploy` 配置.

```shell
deploy:
  type: git
  repo: https://github.com/jtr109/jtr109.github.io
  branch: master 
```

 部署服务到 github:

```shell
hexo clean && hexo deploy
```

## Comment Feature

我参考[这篇文章](https://www.jianshu.com/p/d68de067ea74)进行配置, 写得非常详细. 不多做赘述.

## Favicon

Next theme 默认生成的 icon 比较朴素, 可以使用 [favicon generator](https://favicon.io/favicon-generator/) 生成自己的 favicon, 既可以享受定制化的乐趣, 也避免了版权纠纷.

![favicon generator | favicon.io](/images/hexo-typora-github.assets/image-20191109115147241.png)

点击右上角 Download 下载到本地, 解压并复制到 `themes/next/source/images` 下, 并修改 Next 的配置文件 `nvim themes/next/_config.yml`.

```yml
favicon:
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
```

## 添加分类

添加分类可以帮助我们管理自己的文章, 使用方式可参考[这篇博客文章](https://whx4j8.github.io/2016/03/16/hexo-next-%E6%B7%BB%E5%8A%A0%E4%B8%BA%E6%96%87%E7%AB%A0%E6%B7%BB%E5%8A%A0%E5%88%86%E7%B1%BB/).
