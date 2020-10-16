---
title: "OpenID Connection (OIDC) 原理实践"
date: 2020-08-06T14:04:44+08:00
typora-root-url: ../../static
categories:
  - tect
series:
tags:
  - OpenID
  - OIDC
  - authentication
---

## 流程概览

![](/images/oidc.assets/Sequence_Diagram1.jpg)

## 名词解释

- end user (EU)：实施操作的用户
- relying party (RP)：希望登录的网站
- OpenID provider (OP)：身份信息认证服务
- ID token：用户身份票据，格式为 JWT，包含了用户在 OP 中的 ID，通过 `sub` 字段描述。
- access token：亦称为 token，是 EU 授权给 RP 的票据，RP 可以通过 access token 获取用户已授权的存储在 OP 中的信息。
- nonce：RP发送请求的时候提供的随机字符串，用来减缓重放攻击，也可以来关联 ID token 和 RP 本身的 session 信息。
- state：同OAuth2。防止 CSRF, XSRF。
- scope：EU 授权给 RP 的用户信息范围，可选字段见[文档](https://openid.net/specs/openid-connect-basic-1_0.html#rfc.section.2.4)。

## 构建 OpenID Provider

参考 [Django OIDC Provider](https://django-oidc-provider.readthedocs.io/en/latest/index.html) 文档拉起构建 Django 服务。进入管理平台后创建一个 client，需要指定的配置有：

- response type 指定为 `id_token token`
- redirect URLs 可以填写多个，每行填写一个，在向 OpenID provider 请求身份的时候必须携带一个重定向地址，这个地址必须包含在配置中。
- scopes 指定为 `openid` 如果需要 profile 或者 email 信息，也可以指定，例如： `openid profile email`。（⚠️ 注意：目前测试下来，这里的 scope 配置并不影响 relying party 请求授权的 scopes）

![](/images/oidc.assets/Untitled.png)

OIDC 模块在 Django Admin 中提供的配置页面

## Relying Party 获取身份信息

### 请求授权

要求 EU 请求 OP 的 authorize 接口，携带以下参数指定 RP 的身份：

- `response_type`
- `client_id`
- `redirect_uri`
- `state`
- `scope`
- `nonce`

其中 `state` 和 `nonce` 是可选的，用来防止重放攻击。请求地址示例：

```
http://localhost:8000/openid/authorize/?response_type=id_token%20token&client_id=932981&redirect_uri=http%3A%2F%2Flocalhost%3A3000&scope=openid%20profile&state=123123&nonce=n-0S6_WzA2Mj
```

### 获取授权

EU 确认授权后，被要求重定向到之前指定的 `redirect_uri`。同时携带上 OP 提供的参数：

- `access_token`
- `id_token`
- `token_type`
- `expires_in`
- `state`

```
http://localhost:3000/#access_token=3f0a65d53906451393cab6b0f12a8106&id_token=eyJhbGciOiJSUzI1NiIsImtpZCI6IjExNmRhNzAxMDE1ODQyMmZmODM2MTBiMzYzYTA0MGNmIn0.eyJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwMDAvb3BlbmlkIiwic3ViIjoiMSIsImF1ZCI6IjkzMjk4MSIsImV4cCI6MTU5NTg0MTUyNSwiaWF0IjoxNTk1ODQwOTI1LCJhdXRoX3RpbWUiOjE1OTU4MjIxOTUsIm5vbmNlIjoibi0wUzZfV3pBMk1qIiwiYXRfaGFzaCI6IkdadDZOUjUwMElWU3ZqbXZaZGFIbUEifQ.FSEqMu_xOfYlCEXRyB5vvFQPpQKuax6gQ2axKF3AmI-IytX9g1lWpw6q0jppaMkBCNHi5mses_EbEOWMwXVMDndJDuzIhiL6ZvU0e9hPDdSVRkKRVkgEHCqHKmIOmUe6wYdTfhcZANGVzAjFum81jVuTX5wSMvrVIYk5bad7XoKUJ30WlauPqnykmfZG6Jfw_LDzSe10yJm0EPudI1Tc5lSRysZGqUYXkjcpkzTHYIiSv0NVr88As-bLFwQdyTDMD3DPqnGI2Ruasdeo-ewxjEHFJbINndoKN5PNeLQ2OUoXuAx_pwFInQKm4FNYOHb0ogSvu7d6HAq0LGZESJx51Q&token_type=bearer&expires_in=3600&state=123123
```

### 验证 ID Token

提取出参数 `id_token` ，该参数为 JWT 形式，可以在 [jwt.io](https://jwt.io) 中模仿验证过程。首先提取 `id_token`：

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IjExNmRhNzAxMDE1ODQyMmZmODM2MTBiMzYzYTA0MGNmIn0.eyJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwMDAvb3BlbmlkIiwic3ViIjoiMSIsImF1ZCI6IjkzMjk4MSIsImV4cCI6MTU5NTg0MTUyNSwiaWF0IjoxNTk1ODQwOTI1LCJhdXRoX3RpbWUiOjE1OTU4MjIxOTUsIm5vbmNlIjoibi0wUzZfV3pBMk1qIiwiYXRfaGFzaCI6IkdadDZOUjUwMElWU3ZqbXZaZGFIbUEifQ.FSEqMu_xOfYlCEXRyB5vvFQPpQKuax6gQ2axKF3AmI-IytX9g1lWpw6q0jppaMkBCNHi5mses_EbEOWMwXVMDndJDuzIhiL6ZvU0e9hPDdSVRkKRVkgEHCqHKmIOmUe6wYdTfhcZANGVzAjFum81jVuTX5wSMvrVIYk5bad7XoKUJ30WlauPqnykmfZG6Jfw_LDzSe10yJm0EPudI1Tc5lSRysZGqUYXkjcpkzTHYIiSv0NVr88As-bLFwQdyTDMD3DPqnGI2Ruasdeo-ewxjEHFJbINndoKN5PNeLQ2OUoXuAx_pwFInQKm4FNYOHb0ogSvu7d6HAq0LGZESJx51Q
```

可以注意到 jwt.io 中在解析 JWT 后，解析了 ID token 中 `iss` 指定的地址，向 `.well-known/openid-configuration` 地址发送请求，获取了 JWK 的存放地址 `jwks_uri` 。

![](/images/oidc.assets/Untitled%201.png)

[jwt.io](http://jwt.io) 请求 `iss` 下的 `/.well-known/openid-configuration`

通过请求到的公钥验证了 ID token。

![](/images/oidc.assets/Untitled%202.png)

通过 `iss` 下的 `/jwks` 获取公钥

### 获取用户身份

解析 `id_token` 的 payload 可以获取到鉴权所需的所有信息。其中 `sub` 为 EU 在 OP 中的 ID， `nonce` 为 RP 提供，可以用来验证是否遭到重放。

```json
{
  "iss": "http://localhost:8000/openid",
  "sub": "1",
  "aud": "932981",
  "exp": 1595842560,
  "iat": 1595841960,
  "auth_time": 1595822195,
  "nonce": "n-0S6_WzA2Mj",
  "at_hash": "9EJSyU5K0pBbW1x2FMdbrQ"
}
```

### 获取已授权的用户信息

RP 根据指定的 `token_type` 和 `access_token` 获取被授权的用户信息。

```bash
curl -X GET http://localhost:8000/openid/userinfo -H "Authorization: Bearer 3f0a65d53906451393cab6b0f12a8106" -i
```

![](/images/oidc.assets/Untitled%203.png)

返回已授权的用户信息

### 用户登录 RP

根据以上获取的用户 ID 就可以定位到用户，获取的用户信息可以用作创建或者更新。实际登陆态由 RP 自己控制，如果登录会话过期了，如果在当前网站登录一样，需要回到登录页面，要求 EU 再次授权身份。

## 资料

- [OpenID Connect | Auth0](https://auth0.com/docs/protocols/oidc)
- [OpenID Connect Document](https://openid.net/connect/)（[附译文](https://www.cnblogs.com/linianhui/p/openid-connect-core.html)）
- [OAuth 不能作为身份认证的原因](https://oauth.net/articles/authentication/)（[附译文](https://www.cnblogs.com/linianhui/p/authentication-based-on-oauth2.html)）
- [Harbor 的 OIDC 适配](https://goharbor.io/docs/1.10/administration/configure-authentication/oidc-auth/)
- [JWT.io](https://jwt.io/)