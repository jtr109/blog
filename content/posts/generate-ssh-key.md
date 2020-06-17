---
title: "Generating SSH Key Pairs"
date: 2020-06-17T19:13:42+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - security
---

ED25519 is more secure and recommended to be used as SSH key pairs now. Let's have a try:

```shell
ssh-keygen -t ed25519 -C "<comment>"
```

## References

* https://docs.gitlab.com/ee/ssh/README.html#generating-a-new-ssh-key-pair

