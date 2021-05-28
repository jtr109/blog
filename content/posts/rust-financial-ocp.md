---
title: "使用 Rust 实现数据库对数据模型的依赖"
date: 2020-11-12T16:33:27+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Rust Notes
tags:
  - Rust
  - architecture
---

在学习《架构整洁之道》时，在「开闭原则（OCP）」一章的思想实验设计了如下架构：

![财务数据系统符合开闭原则的架构](/images/rust-financial-ocp.assets/image-20201112163526657.png)

_⬆️ 截图自：孙宇聪. 架构整洁之道 (Chinese Edition) (Kindle Locations 1177-1179). Kindle Edition._

右侧的依赖关系引起了我的注意：Database 组件依赖 Interactor 组件，而非反过来，我尝试使用 Rust 实现上述的依赖关系。

<script src="https://gist-it.appspot.com/https://github.com/jtr109/rust-snippets/blob/master/financial-ocp/src/lib.rs"></script>

