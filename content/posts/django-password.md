---
title: "Django Password"
date: 2019-03-27T13:59:45+08:00
categories:
  - Tech
tags:
  - Django
  - security
draft: false
typora-root-url: ../../static
---

## Set Password Manually

It is easy to set password with the instance of `User` by `set_password`:

```python
user = User.objects.create(username='my_username')
user.set_password('my_password')
```

But sometimes we want to bulk create users of we want to create an `User` model before saving into database. Your can create user model and set password manually.

```python
from django.contrib.auth.hashers import make_password

user = User(
    username='my_username',
    password=make_password('my_password')
)
# now you get the instance of User model
```

## How to Create an Raw Password Automatically?

Django provide a method [`make_random_password`](https://docs.djangoproject.com/en/2.1/topics/auth/customizing/#django.contrib.auth.models.BaseUserManager.make_random_password) in the `BaseUserManager`. So you can generate a password like this.


```python
from django.contrib.auth.models import User

User.objects.make_random_password()  # not a good idea
```

But what if we just want to create a password without `User` involved? You can find the method `make_random_password` just call the function `get_random_string` defined in [`django.utils.crypto`](https://github.com/django/django/blob/master/django/utils/crypto.py). So we make it easy like this.

```python
from django.utils.crypto import get_random_string

raw_password = get_random_string()  # try this one
raw_password = get_random_string(length=16)
raw_password = get_random_string(length=16, allowed_chars='abcdef0123456789')
```

## Reference

- https://docs.djangoproject.com/en/2.1/topics/auth/passwords/#module-django.contrib.auth.hashers
- https://docs.djangoproject.com/en/2.1/topics/auth/customizing/#django.contrib.auth.models.BaseUserManager.make_random_password
