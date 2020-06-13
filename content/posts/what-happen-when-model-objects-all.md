---
title: "What Happen When Model Objects All"
date: 2018-08-27T17:01:12+08:00
draft: false
categories:
  - Tech
tags:
  - Python
typora-root-url: ../../static
---

## Example snippet

```python
from django.db import models

class Product(models.Model):
    pass
```

_create a model_

```python
Product.objects.all()
```

_execute model objects all_

## Consideration

How can we get a queryset from model?

## Source snippet

```python
# django/db/models/base.py

class ModelBase(type):
    def __new__(cls, name, bases, attrs):
        super_new = super(ModelBase, cls).__new__
        # ...
        new_class = super_new(cls, name, bases, new_attrs)
        # ...
        new_class.add_to_class('_meta', Options(meta, app_label))
        # ...
        new_class._prepare()
        # ...
        return new_class

    # ...

    def _prepare(cls):
        # ...

        if not opts.managers or cls._requires_legacy_default_manager():
            if any(f.name == 'objects' for f in opts.fields):
                raise ValueError(
                    "Model %s must specify a custom Manager, because it has a "
                    "field named 'objects'." % cls.__name__
                )
            manager = Manager()
            manager.auto_created = True
            cls.add_to_class('objects', manager)

    def add_to_class(cls, name, value):
        # We should call the contribute_to_class method only if it's bound
        if not inspect.isclass(value) and hasattr(value, 'contribute_to_class'):
            value.contribute_to_class(cls, name)
        else:
            setattr(cls, name, value)

class Model(six.with_metaclass(ModelBase)):
    # ...
```

```python
### django/db/models/manager.py

class BaseManager(object):
    # ...
    def contribute_to_class(self, model, name):
        if not self.name:
            self.name = name
        self.model = model

        setattr(model, name, ManagerDescriptor(self))

        model._meta.add_manager(self)

    # ...

    @classmethod
    def from_queryset(cls, queryset_class, class_name=None):
        if class_name is None:
            class_name = '%sFrom%s' % (cls.__name__, queryset_class.__name__)
        class_dict = {
            '_queryset_class': queryset_class,
        }
        class_dict.update(cls._get_queryset_methods(queryset_class))
        return type(class_name, (cls,), class_dict)

    # ...

    def get_queryset(self):
        """
        Returns a new QuerySet object.  Subclasses can override this method to
        easily customize the behavior of the Manager.
        """
        return self._queryset_class(model=self.model, using=self._db, hints=self._hints)

    def all(self):
        return self.get_request()


class Manager(BaseManager.from_queryset(QuerySet)):
    pass
```

```python
# django/db/models/query.py
class QuerySet(object):
        def __init__(self, model=None, query=None, using=None, hints=None):
            # ...
```

