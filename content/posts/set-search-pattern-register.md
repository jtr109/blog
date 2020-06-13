---
title: 在 Vim 中赋值搜索寄存器
date: 2017-10-25T20:07:41+08:00
categories:
  - tech
tags:
  - Vim
typora-root-url: ../../static
---

## In English

Highlight the search pattern register in Vim.

## 实现的目的

有的时候会遇到这样一个场景. 在当前文件中调用了一个函数, 而这个函数在另一个文件中被定义. 如果有 [jedi](https://github.com/davidhalter/jedi-vim) 之类的神器帮助跳转当然再好不过, 不过如果项目足够复杂, 有的时候 jedi 还是力不从心的. 当这时, 我们的目标可能就变成:

_将光标下的单词能在另一个文件下通过最简单的方式定位到._

## 简单做法

最简单的做法, 当然是在 normal mode 下使用 `*` 把搜索寄存器中的内容更新为光标下的单词. 但是这么做的问题是, 如果该页还有其他匹配项, 就会发生一次跳转. 需要使用 `<c-o>` 跳转回刚刚的位置.

然后 `:b#` 切换到另一个文件 (希望你和我一样设置了映射, 我的是 `<leader-b>`). 文件切换我们这里不做展开.

_这样的做法比较简单, 但是不够优雅._

## 存储到搜索寄存器

那么问题就简单了 -- 怎样可以更新搜索寄存器.

通过 `:h registers` 可以了解到:

```
10. Last search pattern register	"/		*quote_/* *quote/*
Contains the most recent search-pattern.  This is used for "n" and 'hlsearch'.
It is writable with `:let`, you can change it to have 'hlsearch' highlight
other matches without actually searching.  You can't yank or delete into this
register.  The search direction is available in |v:searchforward|.
```

这里可以看到搜索寄存器是可以通过 `:let` 来修改的. 那具体该怎么修改呢? 参见 `:h let-register`

```
:let @{reg-name} = {expr1}			*:let-register* *:let-@*
			Write the result of the expression {expr1} in register
			{reg-name}.  {reg-name} must be a single letter, and
			must be the name of a writable register (see
			|registers|).  "@@" can be used for the unnamed
			register, "@/" for the search pattern.
			If the result of {expr1} ends in a <CR> or <NL>, the
			register will be linewise, otherwise it will be set to
			characterwise.
			This can be used to clear the last search pattern: >
				:let @/ = ""
<			This is different from searching for an empty string,
			that would match everywhere.
```

那就好办了, 手动处理的话用两种方法:

- `:let @/ <C-r><C-w>`. 
- `:let @/='\<'.expand("<cword>").'\>'`


## 将指令保存在 macro 中来调用

建议可以存到 macro 中. 这里也不特意展开了, 给一个例子, 设置 `@m` 为寄存动作:

```
qm  # 打开 m macro 的记录功能
:let @/ <c-r><c-w>  # 或 `:let @/='\<'.expand("<cword>").'\>'`
q  # 关闭记录
```

之后只要调用 `@m` 就可以将当前光标下的文字存储到搜索寄存器中. 一个直观的测试方式, `:set hlsearch` 打开你的搜索高亮, 用 `@m` 试试吧!

## 使用 `nmap` 映射为永久可用的快捷键

但是 macro 有一个问题就是随时可能被替代, 那 map 或许是一个好的解决方式.

```vim
:nnoremap <Leader>m :let @/='\<'.expand("<cword>").'\>'<CR>
# 或者
:nnoremap <Leader>m :let @/='<C-r><C-w>'<CR>
```
