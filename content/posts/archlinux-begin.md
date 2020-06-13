---
title: ArchLinux 安装初探
date: 2017-10-19T22:01:55+08:00
tags:
  - Arch Linux
  - Linux
typora-root-url: ../../static
---

## 前言

本文主要记录了自己在 VirtualBox 中初次成功安装 ArchLinux 的过程. 在回忆的同时再安装一次来保证正确性. 作为笔记的同时, 也为大家提供一个参考.

## 模拟器挂载镜像

首先在 [ArchLinux 官方下载页面](https://www.archlinux.org/download/)下载最新版本的镜像.

打开 [VirtualBox](https://www.virtualbox.org/) 一路确认. 成功生成 virtual machine 后别急着打开, 点击 settings, 在 Storage 中将我们的镜像文件挂载到 Controller: IDE 下. 这里需要注意的是, 默认下载的 torrent 目录名称自带了 `.iso`, 但是我们加载的务必是里面的文件!

设置 network 为桥接

## 安装系统

### 启动 boot 环境

双击加载 machine, 选择 'Boot Arch Linux'.

### 分区

使用 parted 分区

```shell
$ parted
$ mklabel msdos
$ mkpart primary ext4 1M 100M
$ set 1 boot on
$ mkpart primary linux-swap 100M 1.1G
$ mkpart primary ext4 1.1G -1
$ p
$ q
```

### 格式化

```shell
$ lsblk  # 打印设备查看
$ mkfs.ext4 /dev/sda1
$ mkswap /dev/sda2
$ mkfs.ext4 /dev/sda3
$ swapon /dev/sda2
$ mount /dev/sda3 /mnt
$ cd /mnt
$ mkdir boot
$ mount /dev/sda1 /mnt/boot
```

### 修改镜像源

```shell
$ nano /etc/pacman.d/mirrorlist  # 默认就是163的, 可以不修改
$ pacman -Syy  # 更新软件
$ pacstrap /mnt base base-devel
$ genfstab -U /mnt >> /mnt/etc/fstab
```

### 配置 ArchLinux

```shell
$ arch-chroot /mnt /bin/bash
$ pacman -S grub
$ grub-install /dev/sda
$ grub-mkconfig -o /boot/grub/grub.cfg
$ exit  # 返回 root
$ reboot
```

### 开启网络

```shell
$ systemctl enable dhcpcd
```

### 用户设置

```
$ passwd
$ useradd -m <username>
$ passwd <username>
$ visudo
```

#### 设置用户权限

```shell
<username> ALL=(ALL) ALL
```

### 配置镜像

参考[Archlinux-cn镜像使用帮助](http://mirrors.163.com/.help/archlinux-cn.html)以及[Archlinux镜像使用帮助](http://mirrors.163.com/.help/archlinux.html)两篇文章做配置.

其中 `/etc/pacman.conf`:

```shell
[archlinuxcn]
SigLevel = Optional TrustedAll
Server = http://mirrors.163.com/archlinux-cn/$arch
```

`/etc/pacman.d/mirrorlist`:

```shell
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
```

### 安装 yaourt 和 vim

```shell
$ pacman -Syu
$ pacman -S yaourt
$ yaourt -S vim
$ yaourt -S bash-completion
$ yaourt -S net-tools  # 提供 ifconfig 命令
```


### 配置语言环境

```shell
$ vim /etc/locale.gen  # 取消 en_US.UTF-8 的注释
$ locale-gen
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
```

### 配置时区

```shell
$ timedatectl set-timezone Asia/Shanghai
$ date  # 确认时间
```

### 配置主机名

```shell
$echo myarch > /etc/hostname  # 没意思, 可不改
```

### 图形界面

```shell
$ yaourt -s xf86-video-  # 搜索可用的显卡驱动, 对于 vbox 而言就是 vm 打头的. 找到对应的并下载
$ yaourt -S xorg xorg-xinit dbus
$ yaourt -Syu gnome gnome-extra
$ yaourt -S gdm
$ yaourt -S wqy-zenhei  # 中文字体
$ systemctl enable gdm.service  # 开机启动 gdm
```


## 参考

- [Arch Linux 怎么安装？ - 回答作者: 罗然](https://zhihu.com/question/21427410/answer/68411166)
- [细说linux挂载——mount，及其他……](https://forum.ubuntu.com.cn/viewtopic.php?f=120&t=257333): 好友 [@hsyyf](https://github.com/hsyyf) 推荐的文章


## 其他笔记

### 安装的基本思路

1. 调整网络和键盘配置（一般不需要）
2. 分区
3. 挂载分区
4. 修改软件源地址
5. 安装基础包
6. chroot到安装后的系统
7. 修改基础配置文件
8. 安装引导程序
9. 配置引导程序
10. 退出安装环境
11. 重启
12. 卸载iso
13. 进入系统
14. 添加日常用户
15. 分配root权限
16. 安装yaourt（不是必须，但是非常推荐）
17. 安装显卡驱动
18. 安装xorg组件
19. 安装桌面

### yaourt

- `-Syy`: 同步数据库
- `Syu`: 是同步数据库并升级
- `Qdt`: 清理没用的孤包
- `Rns <软件名>`: 可以卸载软件，并且清理依赖软件

### 更新 pacman key

```shell
$ rm -rf /etc/pacman.d/gnupg
$ sudo pacman-key --init
$ sudo pacman-key --refresh-keys
```
