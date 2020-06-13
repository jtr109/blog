---
title: 在 Git 托管服务中使用两步验证能力
categories:
  - tech
date: 2019-11-14 14:06:16
tags:
  - Git
  - Gitlab
  - Security
typora-root-url: ../../static
---

## 写在最前

作为开发者, 应该清楚地知道 "密码再强也不是不可能被破解", 所以只要网站支持 2FA, 就建议在自己的帐号中配置 2FA 支持.

## 在 GitLab 中使用 2FA

GitLab 中如何开启 2FA 在[文档](https://docs.gitlab.com/ee/user/profile/account/two_factor_authentication.html#enable-2fa-via-one-time-password-authenticator)中写得比较清晰, 而且其中推荐了几种可选的 one time password authenticators.

但是按照这个方案操作完之后, 如果你的项目通过 https 指定远程 git 仓库地址, 你的 git 终端会要求你重新登录. 如果你足够耐心, 其实可以在页面下方找到[解决方案](https://docs.gitlab.com/ce/user/profile/account/two_factor_authentication.html#personal-access-tokens).

> When 2FA is enabled, you can no longer use your normal account password to authenticate with Git over HTTPS on the command line or when using [GitLab’s API](https://docs.gitlab.com/ee/api/README.html). You must use a [personal access token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) instead.

所以使用 access token 作为密码即可登录自己的 git 帐号拉取和推送服务.

## 关于 Access Token

建议所有在使用 access token 的过程中, 为每一个客户端使用一个单独的 token, 来避免找到一个 token 变成 "死 token", 即这个 token "可能" 被用在很多地方, 而自己不知道具体在哪使用了.

给每个客户端建立一个 token, 也不要让这些 token 落盘, 如果 logout 了之后, 请刷新 token, 并使用你的新 token 登录.
