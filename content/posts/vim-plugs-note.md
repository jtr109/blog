---
title: vim 插件配置笔记
date: 2017-08-13T13:20:52+08:00
categories:
  - Tech
tags:
  - Vim
typora-root-url: ../../static
---

## 1. 使用技巧

## 2. VIM 配置

### 2.1 `vim-airline`

#### 2.1.1 airline 显示问题

在安装 airline 的过程中, 你很有可能会和我一样踩到很多坑, 这里就和大家分享一下我的经验.

##### 安装成功后 airline 未显示

遇到这个问题是因为没有为 airline 配置设施的行数, 请在 .vimrc 中做如下配置:

```bash
set laststatus=2
```

##### airline 箭头, 符号显示乱码

airline 的乱码问题主要是由于字体问题, 需要通过安装字体和配置 .vimrc 解决. 笔者在安装的过程中也遇到了这个问题, 寻找半天, 终于找到了最平滑易懂的解决办法. 编辑并与大家分享:

作者：[八云](https://www.zhihu.com/question/25219546/answer/45554467)
原文地址: [如何在 Ubuntu 下正确显示 vim-airline 的箭头符号？](http://zhihu.com/question/25219546/answer/45554467)

** 字体补丁安装 **

> - 首先你要安装了补丁字体 [powerline/fonts - GitHub](https://github.com/powerline/fonts), 克隆下来. 
- 运行 `./install.sh`
- 重新设定字体(for gvim)比如 终端也要相应设定(for vim)
`set guifont=DejaVu\ Sans\ Mono\ for\ Powerline\ 11 "config font for gvim" for gvim`

注: 我没有看明白上面最后一句的配置, 所以并没有执行, 但仍达到了效果. 可以把它当做后备方案, 不行再研究. 

> - **用vim 打开** ./vim-airline/doc/airline.txt 目录中的 airline.txt 找到下面的一些语句
- 将其复制到.vimrc中 (例如 `let g:airline_left_sep = ''`)
- 在.vimrc中写上:
  ```
let g:airline_powerline_fonts=1
if !exists('g:airline_symbols')
    let g:airline_symbols = {}
endif
  ```
```
附图:
![airline 图标配置样例]

注: 在 ./vim-airlin/doc/airline.txt 中你可以看到很多款式的的对应符号, 我的建议是全部复制到你的 .vimrc 文件中, 用 `" ` 注释掉重复和不满意的图形(除非你明确的知道自己想要什么).

2.1.2 其他参考:

[安装Vim插件vim-airline](http://www.jianshu.com/p/310368097c75): 这是我最早接触的一篇文章, 不过已经比较久远了, 当时他的下载源还是 bling/vim-airline, 个人更推荐 vim-airline/vim-airline. 不过这篇文章还是有其参考价值的. 文章中提到的 [nerd-fonts](https://github.com/ryanoasis/nerd-fonts) 感觉也是一枚神器, 有兴趣的小伙伴可以了解一下.

2.2 YouCompleteMe

这是一款补全插件, 可以补全 python 等语法, 刚刚开始接触, 对它的特性了解并不多, 不过作为一款神器, 还是将找到的资料和大家分享一下.

2.2.1 参考文章:

[Vim自动补全神器–YouCompleteMe](http://blog.marchtea.com/archives/161)

2.3 综合

[Vim与Python真乃天作之合
](http://codingpy.com/article/vim-and-python-match-in-heaven/): 这篇文章是我重拾 vim 配置的入门参考, 里面分析和介绍了几个比较著名的插件及其作用和安装方法, 可以做个了解.










[airline 图标配置样例]: http://upload-images.jianshu.io/upload_images/2706275-30c29c86ddcda4d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240

```