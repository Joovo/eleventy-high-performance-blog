---
title: 	Look at the memory allocation of Python objects from id function
description:
date: 2019-09-20
tags:

  + python

layout: layouts/post.njk
image:
---
今天偶然发现一个冷知识，之前碰巧没踩到坑:

 `Python 函数的默认值只会在程序加载模块并读取到该函数的定义时设置一次。`

``` python
def A(x=[]):
    return x

a=A()
a.append(1)
>>>[1]

b=A()
b.append(2)
>>>[1,2]
```

在这个例子里 `x` 体现了参数只加载一次的规则，当然一般我们不会这么写，如果一定要初始化列表的话，可以设为 `None` :

``` python
def A(x=None):
    if x is None:
        x=[]
    return x
```

到这里自然联想到另一个问题，不可变类型对象和可变类型对象。数字numbers、字符串string、元组tuples等是不可变类型对象，列表list、字典dict、集合set等是可变类型对象。判断内容是否改变，不能简单根据类型判断。

对于不可变类型对象tuple有一个经典的案例：

``` python
x=[1,2,3]
a=(1,2,3,x)
id(a)
>>> 4318441544
a[3][1]=1111
id(a)
>>>4318441544
```

tuple显然是一个不可变对象，x为变量，list为可变对象。改变过程中x始终没有变，x指向的内容改变，并不影响tuple的不可变性。

上例比较对象我们使用了id，id的官方解释是

```text
id(object)

Return the “identity” of an object. This is an integer which is guaranteed to be unique and constant for this object during its lifetime. Two objects with non-overlapping lifetimes may have the same id() value.

CPython implementation detail: This is the address of the object in memory.
```

简而言之，id是对象的标示，但不是真正的内存地址，真正的内存地址在CPython可见，会存在不同对象的不同的生命周期的id相同的情况。

特例(来自[【Python】Python中的id()和is](https://blog.csdn.net/lnotime/article/details/81194633]))：

``` python
class A:
    def aa(self):
        pass

class B:
    def ba(self):
        pass

    def bb(self):
        pass

    @classmethod
    def bc(cls):
        pass

    @staticmethod
    def bd():
        pass

    def __init__(self):
        pass

a = A()
b = B()
print(id(a.aa))
print(id(b.ba))
print(id(b.bb))
print(id(b.bc))
print(id(b.__init__))
print(id(b.bd))
print(id(b.__hash__))

# 结果：
# 2933781120456
# 2933781120456
# 2933781120456
# 2933781120456
# 2933781120456
# 2933804461728
# 2933804434768
```

每次调用实例的方法，都会使用一块内存，当释放后，下次调用这个实例的其他方法，依然会使用这块内存。

我们知道is是调用id检查的，那么调用 is 呢？

``` python
class B:
    def ba(self):
        pass

    def bb(self):
        pass

b = B()
print(b.ba is b.bb)  # False
print(id(b.ba) == id(b.bb))  # True
```

和直觉是一样的，原因在于is在比较的时候并没有释放一端的内存，当然两个id不一样。同理，如果多个实例的id，也是不同的。
