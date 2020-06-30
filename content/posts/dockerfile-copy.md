---
title: "COPY Command in Dockerfile Keeps Contents in The Distened Folder"
date: 2020-06-30T15:44:46+08:00
typora-root-url: ../../static
categories:
series:
tags:
---

## Problem Description

When I created configures and want to copy them into a docker image based on Nginx.[^1]

```shell
$ tree conf.d
conf.d
└── custom.conf

1 directory, 0 files
```

I wrote a dockerfile shown below and built with it.

```dockerfile
FROM nginx

COPY src/conf.d /etc/nginx/conf.d
RUN ls -l /etc/nginx/conf.d
```

I found the file `/etc/nginx/conf.d/default.conf` which exists in the basic Nginx image was not removed in the built one.

```shell
$ docker build --no-cache .
Sending build context to Docker daemon  4.608kB
Step 1/3 : FROM nginx
 ---> 2622e6cca7eb
Step 2/3 : COPY src/conf.d /etc/nginx/conf.d
 ---> afb504ce8ba6
Step 3/3 : RUN ls -l /etc/nginx/conf.d
 ---> Running in 7aef205e795c
total 8
drwxr-xr-x 2 root root 4096 Jun 30 08:21 custom.conf
-rw-r--r-- 1 root root 1093 May 26 15:01 default.conf
Removing intermediate container 7aef205e795c
 ---> 604a79ae8a67
Successfully built 604a79ae8a67
```

## Explaination

The [document](https://docs.docker.com/engine/reference/builder/#copy) says:

> The `COPY` instruction copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.

The `COPY` command only copies all contents in the source directory but not the source directory itself.

## Solution

The only way to resolve it is set the whole parent directory of `conf.d` as the `<src>` directory. The all contents except `conf.d` will be kept.[^2]

First, let us build our new configure stucture.

```shell
$ tree src
src
└── conf.d
    └── custom.conf
```

Then, change the dockerfile to show both `/etc/nginx` and `/etc/nginx/conf.d` directories.

```dockerfile
FROM nginx

COPY src/conf.d /etc/nginx/conf.d
RUN ls -l /etc/nginx
RUN ls -l /etc/nginx/conf.d
```

When build it with the new dockerfile.

```shell
$ docker build --no-cache .
Sending build context to Docker daemon  3.584kB
Step 1/4 : FROM nginx
 ---> 2622e6cca7eb
Step 2/4 : COPY src/conf.d /etc/nginx/conf.d
 ---> 882c7992707f
Step 3/4 : RUN ls -l /etc/nginx
 ---> Running in d0bf8116ec8a
total 40
drwxr-xr-x 1 root root 4096 Jun 30 08:53 conf.d
-rw-r--r-- 1 root root 1007 May 26 15:00 fastcgi_params
-rw-r--r-- 1 root root 2837 May 26 15:00 koi-utf
-rw-r--r-- 1 root root 2223 May 26 15:00 koi-win
-rw-r--r-- 1 root root 5231 May 26 15:00 mime.types
lrwxrwxrwx 1 root root   22 May 26 15:01 modules -> /usr/lib/nginx/modules
-rw-r--r-- 1 root root  643 May 26 15:01 nginx.conf
-rw-r--r-- 1 root root  636 May 26 15:00 scgi_params
-rw-r--r-- 1 root root  664 May 26 15:00 uwsgi_params
-rw-r--r-- 1 root root 3610 May 26 15:00 win-utf
Removing intermediate container d0bf8116ec8a
 ---> 5626761ee0b8
Step 4/4 : RUN ls -l /etc/nginx/conf.d
 ---> Running in a6597815965f
total 8
drwxr-xr-x 2 root root 4096 Jun 30 08:50 custom.conf
-rw-r--r-- 1 root root 1093 May 26 15:01 default.conf
Removing intermediate container a6597815965f
 ---> c3034a1a51fc
Successfully built c3034a1a51fc
```

We can see all files in `/etc/nginx` original is kept. And the directory `/etc/nginx/conf.d` is totally be overwritten.

By the way, If we want, we can add a new configure file `src/nginx.conf`. It will overwrite the `/etc/nginx/nginx.conf` in new images too.

## References

* [Document of `COPY` command in official site.](https://docs.docker.com/engine/reference/builder/#copy)
* Practices not work for me:
  * https://stackoverflow.com/a/32576340/6522746
  * https://stackoverflow.com/a/30220096/6522746

[^1]: Full example which uses the bad practice [here](https://github.com/jtr109/blog-examples/tree/master/dockerfile-copy/bad-practice).
[^2]: Full example which uses the good practice [here](https://github.com/jtr109/blog-examples/tree/master/dockerfile-copy/good-practice).