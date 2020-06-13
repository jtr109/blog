---
title: "Hashers Issue When Upgrade Django"
date: 2018-09-02T11:11:23+08:00
draft: false
categories:
  - tech
tags:
  - Python
  - Django
typora-root-url: ../../static
---

_Note for `check_password` in Django_

## Background

We are refactoring our project from Django 1.9.10 to a new project with Django 1.11.14 and asyncpg for database connections.

I tried to fork the logic of `User.check_password` and create an async one. But the new one does not work as my _wrong_ exception. But I learn something from that at least.

## Different behavior of `User.check_password` between objects from fixture and created manual

I found when I use the official `check_password` on a user object from fixtures, the value of the object changed. But when I execute it on which I created recently. It did! I was confused about the difference.

_code in `django/contrib/auth/base_user`_

```python
from django.contrib.auth.hashers import (
    check_password, is_password_usable, make_password,
)

# ...

@python_2_unicode_compatible
class AbstractBaseUser(models.Model):
    password = models.CharField(_('password'), max_length=128)

    # ...

    def set_password(self, raw_password):
        self.password = make_password(raw_password)
        self._password = raw_password

    def check_password(self, raw_password):
        """
        Return a boolean of whether the raw_password was correct. Handles
        hashing formats behind the scenes.
        """
        def setter(raw_password):
            self.set_password(raw_password)
            # Password hash upgrades shouldn't be considered password changes.
            self._password = None
            self.save(update_fields=["password"])
        return check_password(raw_password, self.password, setter)
```

_code in `django/contrib/auth/hasher.py`_

```python
def check_password(password, encoded, setter=None, preferred='default'):
    """
    Returns a boolean of whether the raw password matches the three
    part encoded digest.

    If setter is specified, it'll be called when you need to
    regenerate the password.
    """
    if password is None or not is_password_usable(encoded):
        return False

    preferred = get_hasher(preferred)
    hasher = identify_hasher(encoded)

    hasher_changed = hasher.algorithm != preferred.algorithm
    must_update = hasher_changed or preferred.must_update(encoded)
    is_correct = hasher.verify(password, encoded)

    # If the hasher didn't change (we don't protect against enumeration if it
    # does) and the password should get updated, try to close the timing gap
    # between the work factor of the current encoded password and the default
    # work factor.
    if not is_correct and not hasher_changed and must_update:
        hasher.harden_runtime(password, encoded)

    if setter and is_correct and must_update:
        setter(password)
    return is_correct
```

We can find when executed `check_password` of `User` object. The setter will be called when the password `is_correct` and `must_update`. And I find the `must_update` is `False` when checking the one I creatd but `True` in fixtures.

## Why `must_update`?

The variable `preferred` with attribute `must_update` is declared by `get_hasher` and we can find it is an instance of `PBKDF2PasswordHasher` as default. So the dependence of updating is the `iterations` in decoded password.

```python
class PBKDF2PasswordHasher(BasePasswordHasher):

    # ...

    def must_update(self, encoded):
        algorithm, iterations, salt, hash = encoded.split('$', 3)
        return int(iterations) != self.iterations
```

I printed the decoded password and found the iterations in fixtures is 24000 and was changed into 36000 after `check_password`. And here is the truth that the iterations of `PBKDF2PasswordHasher` in version 1.9.10 of Django is 24000 and is changed into 36000 in the version 1.11.14. But of couse it will not make any trouble when `check_password` in the new version, just changing the decoded password of user object.


<details><summary>snippet of `PBKDF2PasswordHasher` in Django 1.9.10</summary>
<p>

```python
class PBKDF2PasswordHasher(BasePasswordHasher):
    """
    Secure password hashing using the PBKDF2 algorithm (recommended)

    Configured to use PBKDF2 + HMAC + SHA256.
    The result is a 64 byte binary string.  Iterations may be changed
    safely but you must rename the algorithm if you change SHA256.
    """
    algorithm = "pbkdf2_sha256"
    iterations = 24000
    digest = hashlib.sha256

    # ...
```

</p>
</details>

<details><summary>snippet of `PBKDF2PasswordHasher` in Django 1.11.14</summary>
<p>

```python
class PBKDF2PasswordHasher(BasePasswordHasher):
    """
    Secure password hashing using the PBKDF2 algorithm (recommended)

    Configured to use PBKDF2 + HMAC + SHA256.
    The result is a 64 byte binary string.  Iterations may be changed
    safely but you must rename the algorithm if you change SHA256.
    """
    algorithm = "pbkdf2_sha256"
    iterations = 36000
    digest = hashlib.sha256

    # ...
```

</p>
</details>

After figuring out the cause of changing in password, I found the tip in the section [Password upgrading](https://docs.djangoproject.com/en/2.1/topics/auth/passwords/#password-upgrading) in Django documentation.

> Hashed passwords will be updated when increasing (or decreasing) the number of PBKDF2 iterations or bcrypt rounds.

## Reference

- [Password upgrading - Django](https://docs.djangoproject.com/en/2.1/topics/auth/passwords/#password-upgrading)
