---
title: "多种 Git Pull 模式差异分析"
date: 2021-05-08T11:37:09+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Git
---

## 背景

在执行 `git pull` 命令时，我们可能会看到这样的提示信息：

```text
hint: Pulling without specifying how to reconcile divergent branches is
hint: discouraged. You can squelch this message by running one of the following
hint: commands sometime before your next pull:
hint: 
hint:   git config pull.rebase false  # merge (the default strategy)
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint: 
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
```

显然 Git 希望我们指定一种 pull 的方式，它提供了三种模式：

* merge
* rebase
* fast-forward only

## 区分

这三种模式都是为了指定 `git pull` 时如果本地和远端代码发生冲突时应该使用什么策略处理。

         A---B---C origin/master
        /
    D---E---F---G master
        ^
        origin/master in your repository

### Merge

Merge 模式会在发生冲突时，将远端分支上的改动 merge 到本地。[官方文档](https://git-scm.com/docs/git-pull#_description)中的实例说明了在触发 `git pull --no-rebase` （merge 模式）之后，变化如下：

```text
          A---B---C origin/master
         /         \
    D---E---F---G---H master
```

可以看到会多出一个 commit H，这个结果可以理解为在本地 master 分支上，执行了 `git merge origin/master`[^warning] 将远端分支合并到了本地分支上。但是可以看到，远端的 master 分支和本地 master 分支还是不同步的。通常情况下这不符合我们的预期。

### Rebase

Rebase 模式和 merge 模式类似，可以理解为在本地 master 分支上执行 `git rebase origin/master`[^warning] 将本地 master 分支基于远端 master 分支进行 rebase。执行完成后，本地的 master 分支会_超前于_远端的 master 分支。

```text
         A---B---C origin/master
        /
    D---E---A---B---C---F'---G' master
```

### Fast-Forward Only

> To phrase that another way, when you try to merge one commit with a commit that can be reached by following the first commit’s history, Git simplifies things by moving the pointer forward because there is no divergent work to merge together — this is called a “fast-forward.”

首先我们需要了解 fast-forward 模式[^ff]，该模式在 `git merge` 中也有，该模式下，如果 Git 可以 resolve the merge，Git 只会移动指针，而不会新建 commit。

加入有下面这样两个分支：

```text
* c5571a5 (HEAD -> master) feat: edit A on master branch
| * b56e7d6 (develop) feat: edit A in develop
|/  
* ea5e820 feat: edit A
* 3af1481 feat: add A
```

即使此时 master 和 develop 分支的结果是一样的，如果我们执行 `git merge develop --ff-only`，会报错：

```text
fatal: Not possible to fast-forward, aborting.
```

我们回到 `git pull`，如果本地和远端代码如下，A 和 A' 中的实际内容是一致的：

```text
     A---B---C origin/master
    /
D---E---A' master
    ^
    origin/master in your repository
```

在执行带有 `--ff-only` 的 `git pull` 命令后，会发生报错。由于两地 commit 不一致，Git 无法通过 fast-forward 实现更新。

下面我们直接看[文档](https://git-scm.com/docs/git-pull#Documentation/git-pull.txt---ff-only)来学习集中 fast-forward 模式：

> With `--ff`, when possible resolve the merge as a fast-forward (only update the branch pointer to match the merged branch; do not create a merge commit). When not possible (when the merged-in history is not a descendant of the current history), create a merge commit.
>
> With `--no-ff`, create a merge commit in all cases, even when the merge could instead be resolved as a fast-forward.
>
> With `--ff-only`, resolve the merge as a fast-forward when possible. When not possible, refuse to merge and exit with a non-zero status.

可以看到，如果对于上面这种情况，我们使用 `--ff`，达到的效果和 `git pull --merge` 是一样的。

[^warning]: 注意！只是打个比方，没有这种操作方式。
[^ff]: 参考[官方文档](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging#:~:text=to phrase that another way%2C when you try to merge one commit with a commit that can be reached by following the first commit's history%2C git simplifies things by moving the pointer forward because there is no divergent work to merge together — this is called a "fast-forward.")。

