---
title: SignalManager in scrapy
description:
date: 2018-08-03
tags:

  + python

layout: layouts/post.njk
image:
---

## Scrapy中的信号量通过SignalManager类实现

``` python
class SignalManager(object):

    def __init__(self, sender=dispatcher.Anonymous):
        self.sender = sender

    def connect(self, receiver, signal, **kwargs):
        """
        Connect a receiver function to a signal.

        The signal can be any object, although Scrapy comes with some
        predefined signals that are documented in the :ref: `topics-signals`
        section.

        :param receiver: the function to be connected
        :type receiver: callable

        :param signal: the signal to connect to
        :type signal: object
        """
        kwargs.setdefault('sender', self.sender)
        return dispatcher.connect(receiver, signal, **kwargs)
```

SignalManager通过PyDispatcher来实现

 `from pydispatch import dispatcher`
PyDispatcher官方文档介绍，PyDispatcher是一个在多个上下文中的多消费者-多生产者-信号处理机制。它主要为了改善包含多个application项目的开发。

### receiver-register(registration)-signal(message)-sender

首先理解 `receiver-register(registration)-signal(message)-sender`
receiver和sender即接受和发送方，接受需要一个注册的过程，定义接受的来源和类型。

### features

摘自官方文档

* 它集中管理分发消息，只要是function（callable objects）就可以作为receiver接受sender的signal
  * 注册时可以为“信任接受”任意sender，特别是 `particular sender` （预定义的sender，如scrapy中），或是匿名 `message` , 即sender为None
  * 注册时可以接受任意signal，特别时 `particular signal` （预定义的siganl，又如scrapy）
  * 单个signal可以被分发到所有可能的receiver，他们彼此间互不干扰
  * 无需为sender或receiver编写额外代码或继承父类（从而使他们变得dispatch-aware）, 任意类型可以为sender，任意callable类型可以为receiver。、
* 可能的话尽量使用对receiver尽量使用弱引用
  * object的生命周期不受注册的行为的影响，当object的生命周期结束时，registration也失效
  * 对实例方法的引用为弱引用
  * weak references can be disabled on a registration-by-registration basis
  * 允许丰富的signal类型，分发过程对signal不做处理
  * 允许发送包含允许被particular receiver处理和其他receiver处理的general messages

``` python
# set up receiver
from pydispatch import dispatcher
SIGNAL = 'my-first-signal'

def handle_event( sender ):
    """Simple event handler"""
    print 'Signal was sent by', sender
dispatcher.connect( handle_event, signal=SIGNAL, sender=dispatcher.Any )

# set up sender
first_sender = object()
# can be any callable object
second_sender = {}
def main( ):
    dispatcher.send( signal=SIGNAL, sender=first_sender )
    dispatcher.send( signal=SIGNAL, sender=second_sender )
```
