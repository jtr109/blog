---
title: Homebrew 镜像推荐
date: 2017-08-18T09:03:47+08:00
categories:
  - tech
tags:
  - Homebrew
typora-root-url: ../../static
---

## 镜像选择

国内比较优秀的镜像源有豆瓣, 清华以及[中科大](https://lug.ustc.edu.cn/wiki/start).

镜像说白了, 只是把国外节点上的文件缓存在国内服务器, 加快国内下载速度. 所以理论上速度都不可能有多大差距, 至于安全性, 就不得而知了.

这里推荐中科大的主要原因是 -- 他家的文档比较清晰, 还有如何重置的说明.

## 镜像地址链接

- [Homebrew Core 源](https://mirrors.ustc.edu.cn/help/homebrew-core.git.html)
- [Homebrew 源](https://mirrors.ustc.edu.cn/help/brew.git.html)
- [Homebrew Bottles 源](https://mirrors.ustc.edu.cn/help/homebrew-bottles.html)
- [Homebrew Cask 源](https://mirrors.ustc.edu.cn/help/homebrew-cask.git.html)

## 简介

Homebrew, core, bottle 和 cask 的作用各不相同, 需要根据需求修改.

- [Homebrew](https://github.com/Homebrew/brew): Homebrew 的整体框架. 对应 `brew upgrade` 下载数据的源, 如果下载其他程序速度慢, 可以考虑修改此源地址.
- [Homebrew Core](https://github.com/Homebrew/homebrew-core): Homebrew 的核心框架, `brew update` 如果速度极慢, 需要修改的就是这个源地址.
- [Homebrew Bottles](https://github.com/Homebrew/brew): 没有使用过, 从文档上看是对应二进制的包, 也就是预编译版本, 感觉没什么用.
- [Homebrew Cask](https://github.com/caskroom/homebrew-cask): 重头戏, 这是用于安装带界面 App 的 tap. 有了它, 你就可以通过 brew 下载和更新这些 App.
- [Homebrew Cask Upgrade](https://github.com/buo/homebrew-cask-upgrade): 如果你开始使用 `brew cask`, 你会发现他没有自带的 upgrade all 方式. 这可以说是一个致命伤, 如果单纯使用 Homebrew Cask, 我们只能先用`$ brew cask outdated` 确认哪些 App 过期了, 再使用 `$ brew cask uninstall` 卸载 App, 然后 `$ brew cask install`重新下载. 而 Homebrew Cask Upgrade 为我们带来的 `$ brew cu` 可以自动化的解决这个问题. 而你需要做的, 只是根据他的提示选择 "yes" or "No".

## 另外

其实镜像只是一个无奈的选择, 从安全性上来说肯定是不如原生源的. 比如使用 homebrew cask 搭配本地镜像源的下载的时候, 我也遇到过 MD5 不一致的情况, 具体原因并没有查明, 但是毕竟有相关的隐患. 所以如果有其他方法可以建立通道, 尽量不要使用国内镜像源. 其实让 Homebrew 走代理的方法也很简单, 下面给出一例.

```bash
$ ALL_PROXY=socks5://127.0.0.1:8000 brew update
# 其他以此类推
```
