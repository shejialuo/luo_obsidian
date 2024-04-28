---
tags:
  - 技术
repository: https:/
---

# Implementation of SUNDR

## Data Structure

![[principle#^fb9c02]]

We first need to define the `principal` base class:

```python
class Principal:
    @property
    def id(self):
        return -1
    def is_user(self):
        return False
    def is_group(self):
        return False
```

There are two types of `principal`, one is `User` and another is `Group` and they will derive the base class `Principal`. The most importatn

