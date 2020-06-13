---
title: Pip Install From Git Repositories
categories:
  - tech
date: 2019-11-27 17:27:14
tags:
  - Python
  - PyPI
  - pip
typora-root-url: ../../static
---

## 包安装

### 问题

在公司项目开发过程中, 我们构建了一些自定义的 Python packages. 如果直接使用如下方式安装, 可以正常使用, 但是 `pip freeze` 的时候只会显示包名称和版本. 无法在实际生产中使用.

```shell
$ pip install git+https://git.example.com/example/foo.git
...
$ pip freeze > requirements.txt
$ cat requirements.txt
dm-utils==1.0
```

### 解决方案

需要参考[这个 issue](https://github.com/pypa/pip/issues/335#issuecomment-1774484)实现:

```shell
$ pip install -e git+https://git.example.com/example/foo.git#egg=0.0.1
$ pip freeze > requirements.txt
$ cat requirements.txt
-e git+https://git.example.com/example/foo.git@2c2ed2b3df2084d1b1ebfa87c9fe703072eded7b#egg=dm_utils
```

### Docker Build

上面的示例可以解决大部分 public 项目的拉取和安装问题, 但如果在 private 项目中使用上面的方案, 在 `docker build` 时会报错, 原因是构建镜像的时候, 镜像中没有我们的 git 用户信息, 无法拉取私有仓库.

需要向项目仓库所有者请求一个 [deploy token](https://docs.gitlab.com/ee/user/project/deploy_tokens/), 使用提供的用户名和 token 执行 `pip install`, 例如:

```shell
$ pip install -e git+https://gitlab+deploy-token-23:KdCjd8-mM1i9DzBZnXTz@git.example.com/example/foo.git#egg=0.0.1
$ pip freeze > requirements.txt
$ cat requirements.txt
-e git+https://gitlab+deploy-token-20:KP9N8J-mM1i9DzBZnXTz@git.example.com/example/foo.git@2c2ed2b3df2084d1b1ebfa87c9fe703072eded7b#egg=dm_utils
```

此时 `docker build` 就不会出现问题.
