---
title: Using Gitlab CI in Project
date: 2019-11-08 09:44:11
categories:
  - tech
tags:
  - Gitlab
  - Python
typora-root-url: ../../static
---

GitLab 的 CI 功能有非常完备的[文档](https://docs.gitlab.com/ee/ci/), 本文旨在介绍一些常用概念和可能遇到的问题, 帮助大家更好的接入和使用.

## 背景

公司项目中有大量 Python 项目构建的微服务, 在合并的过程中需要实现持续集成来获取信息.

## 服务搭建

### 名词解释

* Runner: [Runner](https://docs.gitlab.com/runner/) 是任务的执行单元, 实际上就是一个 OS, 它可以是一个主机, 虚拟机 Docker 或者 Kubernetes.

## Runner 搭建和配置

Shell Runner 有一个麻烦的问题, 就是所有操作都会影响主机的文件, 依赖等, 而且基于主机 OS 的 runner 导致他是独一无二的, 所以无法实现环境区分. 已经 9102 年了, 建议使用 [Docker Runner](https://docs.gitlab.com/runner/install/docker.html).

### 配置 Runner 调度器

工具[文档](https://docs.gitlab.com/runner/install/docker.html#docker-image-installation)安装好 Docker CE 和 `gitlab-runner`, 意味着你拉起了一个 runners 调度器. 

### 配置 Runner

根据[文档](https://docs.gitlab.com/runner/register/index.html#docker)触发 Runner 注册:

```shell
docker run --rm -t -i -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register
```

注册时需要指定 URL 和 registration token, 可以在项目 **Settings -> CI / CD -> Runners** 中查到. 注册时可以指定这个 Runner 的 Tag, 如果项目配置的 Job 中有对应的 Tag, 就会触发该 Runner 的执行.

## Python 集成

以基于 Python 的 CI 配置文件为例, 介绍一下配置的组成. 更加高阶的配置信息可以在[文档](https://docs.gitlab.com/ee/ci/yaml/)中找到.

### 编写配置

```yml
image: python:3.7.4  # 指定使用的基础镜像

job1:
  stage: test
  script:
    - python -c "print('hello')"  # 执行语句
    - python -c "print('world')"
  tags:
    - pyservice  # 指定 runner 对应的 tag
```

需要注意的是 `image`, `before_script` 这些配置既可以作为顶级字段, 也可以在任何一个 job 中单独配置, 优先级从内到外, 即如果 job 内部有 `before_script`, 那这个 job 被触发的时候, 不会读取和执行全局的 `before_script`.

### 设置缓存

在项目实际使用中, 我们会需要引入很多依赖, 如果不做配置, 每次 Runner 执行的时候都会重新下载相关的依赖, 我们可以参考[文档 Caching Python dependencies](https://docs.gitlab.com/ee/ci/caching/#caching-python-dependencies)中的方案进行配置.

另外根据该方案进行配置后, 即使多个 repositories 共用一个 runner, 每个项目的依赖文件也能做到相互隔离, 各自缓存, 互不影响.
