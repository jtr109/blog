---
title: Replace In All Files
date: 2019-11-07 22:40:05
categories:
  - Tech
tags:
  - Vim
  - ripgrep
typora-root-url: ../../static
---
## 背景

在项目开发过程中, 有时需要全局替换一些变量名. 如果使用 VSCode 这类现代编辑器, 可以十分便利地办到. 但是使用 Vim (or NeoVim) 就没有那么容易了.

## 方案

我们需要一个方案来实现全局替换的需求. 需要满足如下要求:

1. 可以自定义查找范围. 包括:

  - 选择根路径
  - 筛选符合指定 pattern 的文件
  - 排除符合指定 pattern 的文件
2. 查看确认筛选结果后, 执行修改

目前想到的办法是通过 `ripgrep` 和 `sed` 实现, 具体方法可以参照 ripgrep 的[文档](https://github.com/BurntSushi/ripgrep/blob/master/FAQ.md#how-can-i-search-and-replace-with-ripgrep).

## 工作流

### 全局查找

```shell
❯ rg "println" .
./src/main.rs
2:    println!("Hello world!");

./backend/src/bin/todo.rs
8:    println!("subcommands");
9:    println!("    new<title>: create a new task");
33:        println!("new: missing <title>");
44:        println!("show: unexpected argument");
50:    println!("TASKS\n-----");
52:        println!("{}|{}|{}", task.id, task.title, task.done);
58:        println!("accomplish: missing <id>");
67:            println!("accomplish: <id> must be i32");
76:        println!("acco: missing <title>");
88:        println!("remove: missing <title>");
```

### 文件名筛选

```shell
❯ rg "println" . --files-with-matches
./src/main.rs
./backend/src/bin/todo.rs
```

### 全局替换

```shell
❯ rg "println" . --files-with-matches -0 | xargs -0 sed -i '' 's/println/pp/g'
```

#tech/shell
