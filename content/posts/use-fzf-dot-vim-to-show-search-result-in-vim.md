---
title: "Use fzf.vim to Show Search Result in Vim"
date: 2019-07-31T11:04:57+08:00
typora-root-url: ../../static
draft: false
categories:
  - tech
series:
  - Vim Notes
tags:
  - Vim
  - fzf
---
## Why fzf.vim

Even though I am a Vimer. I prefer complete solutions, with boundless possible of customization, for coding with Vim (actually NeoVim which I use).

The plugin [fzf.vim](https://github.com/junegunn/fzf.vim) is such a strong performer of them. It can not only help you search files, contents from files, tags or windows, but also provide you features to change the themes of your editor in it just like a GUI editor do.

## Some Searching Tricks

You should spend some time on learning how to use fzf.vim smoothly if you have no experience of using[ fzf](https://www.yuque.com/jtr109/zima/tk6fdc/edit). Now, let me show you some tricks.


### Respecting `.gitignore` 

#### TL;DR

Add this line in your `.vimrc`:

```shell
let $FZF_DEFAULT_COMMAND = 'rg --files'
```

#### Here Is Why

Let's say we have a rust project with `target` directory. When calling `:Files` we find all files even though those in the directory `target` which we expected ignored by specified in `.gitignore`.

![image.png](/images/use-fzf-dot-vim-to-show-search-result-in-vim.assets/1561773303637-969795e1-fa69-46a3-8cca-342e5ef485cd.png)

Because the fzf does not understand the `.gitignore`. Let us reproduct it in command line with fzf.


![image.png](/images/use-fzf-dot-vim-to-show-search-result-in-vim.assets/1561773516508-3e3d3a52-f971-4fe5-9523-0ab84d3d1324.png)

But don't be sad. We can config it to out purpose. Before that, we need a searching engine which respect `.gitignore`, such as [the silver searcher](https://www.yuque.com/jtr109/zima/tk6fdc/edit) or [ripgrep](https://github.com/BurntSushi/ripgrep). Let's use ripgrep (called `rg` as usual) as an example.


We can use the command `rg --files` to list all files without those ignored by `.gitignore`. And we can list them in fzf with `rg --files | fzf`.

Now we just need to specify the fzf.vim use the command `rg --files` to search as default.

```shell
let $FZF_DEFAULT_COMMAND = 'rg --files'
```

Let's have a look at the result now.

![image.png](/images/use-fzf-dot-vim-to-show-search-result-in-vim.assets/1561779008133-7d660fba-b986-411f-81f3-4c265d8a7620.png)

#### By The Way

If you perfer `ag`, try this:

```shell
let $FZF_DEFAULT_COMMAND = 'ag -g ""'
```

## Tips with `rg`
If you use `rg` as default search engine in fzf, you will find all hidden files are excluded. You need to add `--hidden` to show them.

```shell
let $FZF_DEFAULT_COMMAND = 'rg --files --hidden'
```

And now, all hidden files which not ignored by `.gitignore` are shown. But wait a minute, all files in `.git` directory are shown too! Most of time, this is not our purpose. But only the way can solve it is adding `.git` into `.gitignore` directly. Please comment if you find a better way!

### Filter By Content

If we try to search content in your project with fzf using `:Rg`. The result will like below.

![image.png](/images/use-fzf-dot-vim-to-show-search-result-in-vim.assets/1561779585101-a49256e1-834d-4b6d-9af2-cce51a7c66e2.png)

But all results match the pattern are shown even though the results matching the filename. How can we get the results only match with content? Just use `.` in your pattern. The characters before `.` will be known as a pattern for filename and as content pattern after `.`.

Let's have a try which filter only with content, such as `.arg`.

![image.png](/images/use-fzf-dot-vim-to-show-search-result-in-vim.assets/1561779567738-a83a4772-407e-4164-9b03-4be67308de20.png)

What about `arg.` for only filename?

![image.png](/images/use-fzf-dot-vim-to-show-search-result-in-vim.assets/1561779676551-b34a407d-4ccd-4b90-a687-a4cfc8562d4f.png)

## Reference

- [https://github.com/junegunn/fzf.vim/issues/121#issuecomment-209534405](https://github.com/junegunn/fzf.vim/issues/121#issuecomment-209534405)
