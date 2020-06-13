---
title: Export or Not Export in Environment File
categories:
  - Tech
date: 2019-12-10 11:00:25
tags:
  - Env
  - Bash
  - Shell
typora-root-url: ../../static
---

## 背景

在项目中需要使用 `envsubst` 从模板生成配置文件. 发现如果在环境变量文件 `env.conf` (或 `.env` 等) 中不使用 `export` 命令, `envsubst` 无法获取到所需的环境变量.

## 实验

构建模板文件 `src.txt`

    now foo here: ${FOO}

### 不使用 `export`

创建环境变量配置文件 `env.conf`:

```shell
FOO=bar
```

创建执行脚本 `subenv.sh`:

```shell
source env.conf

echo $FOO

envsubst < "src.txt" > "env-tar.txt"
```

执行脚本 `[subenv.sh](http://subenv.sh)` 查看生成的 `env-tar.txt` 文件:

    now foo here:

### 使用 `export`

创建环境变量配置文件 `exp.conf`

```shell
export FOO=bar
```

创建执行脚本 `subexp.sh`:

```shell
source exp.conf

echo $FOO

envsubst < "src.txt" > "exp-tar.txt"
```

执行脚本 `[subexp.sh](http://subexp.sh)` 查看生成文件 `exp-tar.txt`:

    now foo here: bar

*可以看到, 在环境变量配置文件中使用 `export`, 在脚本中使用 `envsubst` 才能获取到所需的环境变量.*

### 其他发现

如果使用上面的脚本测试时, 可以发现无论是否使用 `export`, `echo` 都能正确地拿到所需的环境变量.

## 是否应该在配置文件中使用 `export`

在配置文件中是否应该使用 `export`, 也存在着不同的声音. 在一个[相关高票回答](https://stackoverflow.com/a/19331521/6522746)下, 就有这么一个评论:

> If using this .env file between systems, inserting `export` would break it for things like Java, SystemD, or other tools

且不关心什么方案是更好的实践, 我们希望能找到一个通用的方案.

在[这个回答](https://stackoverflow.com/a/30969768/6522746)中, 我们找到了目前最好的解决方案:

> `-o allexport` enables all following variable definitions to be exported. `+o allexport` disables this feature.

```shell
set -o allexport
source conf-file
set +o allexport
```

经实践, 如果我们将 `subenv.sh` 改造成如下:

```shell
set -o allexport
source env.conf
set +o allexport

echo $FOO

envsubst < "src.txt" > "env-tar.txt"
```

即可在输出文件中获得符合期待的结果:

```txt
new foo here: bar
```

## 相关资料

[Difference between set, export and env in bash](https://hackjutsu.com/2016/08/04/Difference%20between%20set)