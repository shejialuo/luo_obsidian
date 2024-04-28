# Data Structure

![[principle#^fb9c02]]

We first need to create a class `Principal` to represent the  principal.

```python
class Principal:
    @property
    def id(self) -> int:
        raise NotImplementedError

    def is_user(self) -> bool:
        raise NotImplementedError

    def is_group(self) -> bool:
        raise NotImplementedError
```

There are two types of principal, one is user and another is group. They will both derive the `Principla` class like the following:

```python

```