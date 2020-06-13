---
title: 在 NeoVim 中修改指定文件类型的缩进
date: 2019-11-07 22:41:13
categories:
  - Tech
tags:
  - Vim
  - Neovim
typora-root-url: ../../static
---
## 问题描述
使用 NeoVim 编辑 `.sql` 文件的时候发现默认的换行是一个 8 space 的 tab. 希望能将其改成 4 spaces.

## 问题拆解
这其实会涉及到两个问题:

* Vim (NeoVim) 中如何修改缩进
* 如何对指定文件类型修改配置

## 修改缩进
关于修改缩进的需要做几个配置, 在[这篇提问](https://vi.stackexchange.com/questions/13080/setting-tab-to-2-spaces)中可以找到解答:

```vim
filetype plugin indent on
" On pressing tab, insert 2 spaces
set expandtab
" show existing tab with 2 spaces width
set tabstop=2
set softtabstop=2
" when indenting with '>', use 2 spaces width
set shiftwidth=2
```

我们只要将 `2` 改成 `4` 就能符合我们的要求. 但是还有一个问题: 如果按照这样修改, 会全局修改我们的缩进规则, 而我们只希望这个配置在指定类型的文件中生效.

## 查看文件类型
为了对指定文件做配置, 我们首先需要知道 Vim 认为当前文件属于什么类型. 可以在 Normal 状态下使用命令:

```vim
:set filetype?
```

如果在 `.sql` 文件中使用命令, 会得到结果:

```vim
filetype=sql
```

即 `filetype` 是 `sql`

## 配置指定文件类型
在 Vim 的配置目录 `~/.config/nvim` (Vim为 `~/.vimrc`) 下新建 `ftplugin` 目录, 根据文件类型 (上面示例中为 `sql`) 新建配置文件, 在其中添加配置.

这里需要注意的是, 配置参数时使用 `setlocal` 而非 `set` 来控制影响范围. 具体用法请参考[这篇回答](https://vi.stackexchange.com/a/9435).

所以我们需要在 `~/.config/nvim/ftplugin/sql.vim` 中写入配置:

```vim
" On pressing tab, insert 4 spaces
setlocal expandtab
" show existing tab with 4 spaces width
setlocal tabstop=4
setlocal softtabstop=4
" when indenting with '>', use 4 spaces width
setlocal shiftwidth=4
```

## 重载编辑器
如果想在不关闭编辑器的情况下重载, 可以使用 `source` 命令加载主配置文件, 已 `~/.config/nvim/init.vim` 为例:

```vim
source ~/.config/nvim/init.vim
```


#tech/vim