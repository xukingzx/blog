---
title: pytho函数式编程相关
date: 2019-07-29 17:02:14
tags:
---
## 装饰器技术

## 装饰器修复技术@wraps
Python装饰器（decorator）在实现的时候，被装饰后的函数其实已经是另外一个函数了（函数名等函数属性会发生改变），为了不影响，Python的functools包中提供了一个叫wraps的decorator来消除这样的副作用。写一个decorator的时候，最好在实现之前加上functools的wrap，它能保留原有函数的名称和docstring。

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        """decorator"""
        print('Calling decorated function...')
        return func(*args, **kwargs)

    return wrapper


@my_decorator
def example():
    """Docstring"""
    print('Called example function')


def normallize(name):
    return name[0:1].upper() + name[1:].lower()


print(example.__name__, example.__doc__)
```

执行结果：
wrapper decorator

```python
from functools import wraps


def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        """decorator"""
        print('Calling decorated function...')
        return func(*args, **kwargs)

    return wrapper


@my_decorator
def example():
    """Docstring"""
    print('Called example function')


def normallize(name):
    return name[0:1].upper() + name[1:].lower()


print(example.__name__, example.__doc__)
```

执行结果：
example Docstring

1. Why 为什么有这个技术
2. What 这个技术解决了什么问题
3. How 这个技术是怎么使用的
