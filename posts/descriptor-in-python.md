---
title: descriptor in python
description:
date: 2019-09-19
tags:

  + python

layout: layouts/post.njk
image:
---

## 描述符 descriptor

描述符和属性调用，元编程有密切的关系，它还是property，classmethods，staticmethods的实现原理。

实现了描述符协议 `descriptor protocol` 的类是一个描述符 `descriptor` 。

``` python
Descriptor Protocol
descr.__get__(self, obj, type=None) -> value

descr.__set__(self, obj, value) -> None

descr.__delete__(self, obj) -> None
```

实现三者任意其一的即满足描述符协议。根据是否实现get和set，可分为数据描述符 `data descriptor` 和非数据描述符 `non-data descriptor` 。

除了封装数据的常见用法，另外可用于隐藏类的内容，只开放get和set，del作为接口：

* 内省描述符：在 `__set__` 或 `__get__` 中完成自我检查，如生成、检查 `doc` 是否合法
* 元描述符：使用宿主的一个或多个方法来执行一个任务，用于元编程，在代码运行中动态绑定实例的方法。注册 `data descriptor` 来更新原来的 `__dict__` 或 `non-data descriptor` 实现 `monkey patch`
详见《Python高级编程》第一版p71

### 从属性调用上讲起

对于一个类

``` python
# descriptor protocol class, as a descriptor in class A
class d:
    def __get__(self,obj,obj_type):
        print('call d.__get__')
        pass

# simple class
class A:
    x=d() # non-data desciptor
    y=5 # will be in __dict__

a=A()
```

通过 `a.__getattribute__('x')` 调用属性如 `a.x` 。只有当 `raise AttributeError` （可能是 `__getattribute__` 也可能是 `__get__` 产生的 `property` 只读抛出）才调用 `__getattr__` , `getattr()` 则是内建函数，和 `setattr` , `hasattr` 关系密切。

1. 先检查x是否为 `data descriptor`
2. 其次查找 `__dict__()` （所有元素包括属性、方法，函数为 `dict` , 类为只读 `mappingproxy` ，这是题外话）
3. 最后检查 `non-data descriptor`
只有一个方法，但是查找3次，有点奇怪，但实际上写一个demo，分别给x绑定三种类型，能体现查找顺序，这是实现 `monkey-patch` 的底层基础之一。

上例中 `a.x` 将调用 `d.__get__` 。

### 对于 `descriptor` 的调用

原文：

```python
For objects, the machinery is in object.__getattribute__() which transforms b.x into type(b).__dict__['x'].__get__(b, type(b)).

b.x <=> type(b).__dict__['x'].__get__(b,type(b))

For classes, the machinery is in type.__getattribute__() which transforms B.x into B.__dict__['x'].__get__(None, B)

B.x <=> B.__dict__['x'].__get__(None,B)
```

`<Descriptor type>.__get__(*args)` ，实例和类的参数不同

### 一个应用 `property`

官方 `Howto` 文档用py来实现property(限制读写访问)的例子。

``` python
class Property(object):
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        print('call getter')
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        print('call setter')
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)

# 对于这个类，查看调用的方法，并没有起限制作用
class C:
    def __init__(self):
        self._x = None

    def getx(self):
        print('call getx')
        return self._x

    def setx(self, value):
        print('call setx, value=%s' % value)
        self._x = value

    def delx(self):
        del self._x

    x = Property(getx, setx, delx, "I'm the 'x' property.")

c=C()
c.x
# call getx 通过__get__调用了fget,即 getx，具体是 getx(c)
c.x = 5
# call setx 通过__set__调用了fset,即 setx(c，5)
```

来看一个真正的例子：

``` python
class C:
    '''
    对x方法进行限制读写，readonly
    '''
    def __init__(self):
        self._x = None

    @property
    def x(self):
        """I'm the 'x' property."""
        return self._x

c=c()
c.x=5 # AttributeError
```

调用@property装饰器将x方法注册到fget中，c.x可以通过__get__调用fget, 而调用赋值语句c.x=5。由于fset方法没有被重写/注册，通过__set__找不到fset（为None）。

如果要添加setter, deleter方法，则添加

``` python
'''
x是property装饰后的property类，
查看propery.setter和deleter的代码，和初始化类似，也即和property装饰器第一次装饰的过程类似。
'''
@x.setter
    def x(self, value):
        self._x = value

@x.deleter
def x(self):
    del self._x
```

|||
|----|----|
|WeChat| Jo_0525|
|Gmail|wohfacm@gmail.com|
|163| wohfacm@163.com|
