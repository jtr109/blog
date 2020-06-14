---
title: "Set Environment Variables in Vim"
date: 2019-06-29T11:54:22+08:00
typora-root-url: ../../static
draft: false
categories:
  - tech
series:
  - Vim Notes
tags:
  - Vim
---
Sometimes, we want to set an environment variable in Vim without contaminating the environment variable in our system.

You can try setting it only in `.vimrc` like below.

```shell
let $FZF_DEFAULT_COMMAND = 'rg --files'
```

