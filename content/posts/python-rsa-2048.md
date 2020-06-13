---
title: "在 Python 中使用 RSA 2048"
date: 2020-06-10T14:54:41+08:00
typora-root-url: ../../static
draft: false
tags:
  - rsa
  - cryptographic
---

## 使用 OpenSSL 生成公私钥对

生成私钥：

```shell
 openssl genrsa -out private_key.pem 2048
```

通过私钥生成公钥：

```sh
openssl rsa -in private_key.pem -outform PEM -pubout -out public.pem
```

## 使用 Python 生成公私钥对

参考[官方文档](https://pycryptodome.readthedocs.io/en/latest/src/examples.html#generate-public-key-and-private-key)

```python
from Crypto.PublicKey import RSA

key = RSA.generate(2048)
private_key = key.export_key()
file_out = open("private.pem", "wb")
file_out.write(private_key)
file_out.close()

public_key = key.publickey().export_key()
file_out = open("receiver.pem", "wb")
file_out.write(public_key)
file_out.close()
```



## 使用 Python 加解密

使用 [PyCryptodome](https://pycryptodome.readthedocs.io/en/latest/index.html) 实现 RSA-2048 加解密。

```python
In [1]: from Crypto.PublicKey import RSA

In [2]: from Crypto.Cipher import PKCS1_OAEP

In [3]: private_key = RSA.import_key(open("/Users/jtr109/Downloads/private_key.pem").read())

In [4]: public_key = RSA.import_key(open("/Users/jtr109/Downloads/public_key.pem").read())

In [5]: encryptor = PKCS1_OAEP.new(public_key)

In [6]: data = b'Hello world'

In [7]: encrypted_data = encryptor.encrypt(data)

In [8]: decryptor = PKCS1_OAEP.new(private_key)

In [9]: decryptor.decrypt(encrypted_data)
Out[9]: b'Hello world'
```

## FAQ

### 可以使用私钥加密，公钥解密吗？

公钥是对外复数发放的，有不止一个对象可以获得公钥。

一般的操作方式是公钥加密、私钥解密。另一种用法是私钥签名、公钥验证。
