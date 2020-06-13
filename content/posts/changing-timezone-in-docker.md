---
title: "Changing Timezone in Docker"
date: 2019-06-17T11:40:44+08:00
typora-root-url: ../../static
draft: false
categories:
  - tech
series:
tags:
  - time zone
  - Docker
---
## Background

I created a script and it ran well in my local environment. But after it was built in the docker container the string of time was not shown as I expected.

## Solution

The difference of timezone between the docker container and host make the problem. Time is always a problem in digital world it always causes problems. So let us resolve it in docker container with dockerfile.


- changing the environment value `TZ`
- changing localtime and timezone in image


```shell
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

<script src="https://gist.github.com/jtr109/21cb7ef7c79034b2ad63df8afb03e7cb.js"></script>

## Note

When I searched the solution, I found some difference in alpine base images.


> Note: if you are using an alpine based image you have to install the `tzdata` first. (see this issue [here](https://github.com/gliderlabs/docker-alpine/issues/136))



Looks like this:
```
RUN apk add --no-cache tzdata
ENV TZ America/Los_Angeles
```

## Reference

- [Changing Timezone in Dockerfile](https://serverfault.com/a/683651)
- [Five Ways to Change Time in Docker Container](https://bobcares.com/blog/change-time-in-docker-container/)

