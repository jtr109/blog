---
title: Mysql Dump 笔记
date: 2017-08-13T14:04:20+08:00
tags:
  - mysql
typora-root-url: ../../static
---

目前的配置环境均为 Windows，其他环境会在稍后补充

## 1. 全量备份


## 2. 增量备份及恢复

### 2.1 备份

#### 2.1.1 启用二进制备份

##### 开启 binlog 日志

``` bash
mysql -hlocalhost -uroot -pjue -e "set global sql_log_bin=1";  
mysql -hlocalhost -uroot -pjue -e "show global variables like 'sql_log_bin'\G";  
```

实测需要将 global 替换为 session

##### 在系统 MySQL 配置文件 my.ini 中加入（修改）存储 log-bin 的位置

```bash
log-bin=D:/Program Files (x86)/MySQL/mylog/jlog
```

#### 2.1.2 重启 MySQL 服务

#### 2.1.3 执行一次全量备份

```bash
$ mysqldump -hlocalhost -uroot -p -l -F test > testtt.sql
```

全量备份除了新建 sql 备份意外，还会新建一个 binlog 文件，完成后可以删除早先的 binlog 文件。
转移全量备份的 sql 并保存。

#### 2.1.4 定期备份 binlog 文件

##### 拆分 binlog 文件

```bash
$ mysql -hlocalhost -uroot -p -e "flush logs;"
```

执行语句会生成新的 binlog 文件用于记录， 原先的 binlog 文件可以转移并保存


### 2.2 恢复

### 2.2.1 单增量恢复

找到全量恢复之前的所有 binlog 文件， 同时导入

```bash
$ mysqlbinlog binlog.000014 binlog.000015 | mysql -uroot -p
```

如果出现

```bash
ERROR 1062 (23000) at line 35: Duplicate entry '6' for key 'PRIMARY'
```

则需要重新导入全量

### 2.2.2 复合导入

```bash
# 导入全量
$ mysql -hlocalhost -uroot -p test < testtt.sql
# 增量导入
$ mysqlbinlog binlog.000014 binlog.000015 | mysql -uroot -p
```

这样的导入方法只能完整地导入整个 binlog 文件中的数据， 我们还可以加入参数来完成部分导入：

```bash
# 根据时间导入
```


### 2.3 问题汇总

#### 2.3.1 MySQL配置文件 my.ini 位置

打开“服务”，找到 MySQL 相关服务（如 MySQL57），点击属性。“可执行文件的路径”的显示即为 my.ini 位置。

## 3. 参考文档

### 3.1 全量备份

- [【SQL】MySQL之使用mysqldump全备份及恢复过程详解](http://blog.csdn.net/jueblog/article/details/9878191)

### 3.2 增量备份

- [【SQL】MySQL之使用mysqlbinlog进行增量备份及恢复详解](http://blog.csdn.net/jueblog/article/details/9909669)
- [MySQL增量备份与恢复实例](https://segmentfault.com/a/1190000002408168)
