---
title: pytho函数式编程相关
date: 2019-07-29 17:02:14
tags:
---
## 从一段看不懂的代码说起
在python3-cookbook学习内联回调函数这一节时，样例代码(如下)看不懂，回去补了下装饰器技术，wraps注解的作用，yield的作用等相关内容。
```python
from queue import Queue
from functools import wraps


def apply_async(func, args, *, callback):
    result = func(*args)
    callback(result)


class Async:
    def __init__(self, func, args):
        self.func = func
        self.args = args


def inlined_async(func):
    @wraps(func)
    def wrapper(*args):
        f = func(*args)
        result_queue = Queue()
        result_queue.put(None)
        while True:
            result = result_queue.get()
            try:
                a = f.send(result)
                apply_async(a.func, a.args, callback=result_queue.put)
            except StopIteration:
                break

    return wrapper


def add(x, y):
    return x + y


@inlined_async
def test():
    r = yield Async(add, (2, 3))
    print(r)
    r = yield Async(add, ('Hello', 'World'))
    print(r)
    for n in range(10):
        r = yield Async(add, (n, n))
        print(r)
    print('GoodBye')


if __name__ == '__main__':
    import multiprocessing
    pool = multiprocessing.Pool()
    apply_async = pool.apply_async
    test()
```
结果：
```python
5
HelloWorld
0
2
4
6
8
10
12
14
16
18
GoodBye
```
## 装饰器技术
### 无参装饰器
```python
def use_logging(func):
    def wrapper(*args, **kwargs):
        print(args)
        print(kwargs)
        print('%s is running' % func.__name__)
        return func(*args, **kwargs)

    return wrapper


@use_logging
def bar(x):
    print('I am bar %d' % x)


if __name__ == '__main__':
    # bar = use_logging(bar)
    bar(x=2)
    print(bar.__name__)
```
输出：
```python
()
{'x': 2}
bar is running
I am bar 2
wrapper
```
可见@use_logging的实际的工作就是bar = use_logging(bar)
### 有参装饰器
```python
def use_logging(level):
    if level == 'warn':
        def decorator(func):
            def wraper(*args, **kwargs):
                print('adfds')
                return func(*args)

            return wraper

        return decorator
    else:
        def decorator(func):
            print('wx')
            return func

        return decorator


@use_logging(level='error')
def foo(name='foo'):
    print('I an %s' % name)


if __name__ == '__main__':
    foo()
```
输出：
```python
wx
I an foo
```
1. 如果直接对原有函数改动，违反开闭原则，引入装饰器，可以在新增额外功能的时候只是新增一个函数，而不是修改原来的代码
2. 可以通过装饰器，在对原函数不做改动的情况下，增加额外的功能
3. 装饰器是定义一个入参为函数的函数，然后通过在原有函数上面添加注解的方式，增加新功能。
4. Q:如上代码，为什么wraper函数能够获取原有函数的入参

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
```python
wrapper decorator
```

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
```python
example Docstring
```

1. Why 为什么有这个技术
2. What 这个技术解决了什么问题
3. How 这个技术是怎么使用的
