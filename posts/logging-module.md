---
title: 	logging module in Python
description:
date: 2019-08-08
tags:
  - python
layout: layouts/post.njk
image:
---


## intro

Python 的 logging 模块，经常在包的源码中看到。

``` python
日志级别依次递增，默认只显示 warning 级别的日志。
import logging
logging.debug('debug message')
logging.info('info message')
logging.warning('warning message')
logging.error('error message')
logging.critical('critical message')
```

## 配置 Config

``` python
logging.basicConfig(level=logging.DEBUG, # 级别
                    format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s', # 日志格式
                    datefmt='%Y-%m-%d %H:%M:%S',  # 时间格式
                    filename='/tmp/test.log', # 保存地点
                    filemode='w') # 访问模式
```

format的参数：

``` python
%(name)s Logger的名字
%(levelno)s 数字形式的日志级别
%(levelname)s 文本形式的日志级别
%(pathname)s 调用日志输出函数的模块的完整路径名，可能没有
%(filename)s 调用日志输出函数的模块的文件名
%(module)s 调用日志输出函数的模块名
%(funcName)s 调用日志输出函数的函数名
%(lineno)d 调用日志输出函数的语句所在的代码行
%(created)f 当前时间，用UNIX标准的表示时间的浮 点数表示
%(relativeCreated)d 输出日志信息时的，自Logger创建以 来的毫秒数
%(asctime)s 字符串形式的当前时间。默认格式是 “2003-07-08 16:49:45,896”。逗号后面的是毫秒
%(thread)d 线程ID。可能没有
%(threadName)s 线程名。可能没有
%(process)d 进程ID。可能没有
%(message)s用户输出的消息
```

## 进阶使用

### 创建logging

在模块中经常会看到 `LOG=logging.getLogger(__main__)` 或者 `LOG=logging.getLogger()` 。

``` python
#coding:utf-8
import logging
logger1 = logging.getLogger()
logger1.setLevel(logging.DEBUG)

logger2 = logging.getLogger('mylogger')
logger2.setLevel(logging.INFO)

# 创建一个handler，用于写入日志文件
fh = logging.FileHandler('/tmp/test.log')

# 再创建一个handler，用于输出到控制台
sh = logging.StreamHandler()

# 设置格式
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

fh.setFormatter(formatter)
sh.setFormatter(formatter)

# 添加handler
logger1.addHandler(fh)
logger1.addHandler(ch)

logger2.addHandler(fh)
logger2.addHandler(ch)

```

创建日志

``` python

logger1.debug('logger1 debug message')
logger1.info('logger1 info message')
logger1.warning('logger1 warning message')
logger1.error('logger1 error message')
logger1.critical('logger1 critical message')

logger2.debug('logger2 debug message')
logger2.info('logger2 info message')
logger2.warning('logger2 warning message')
logger2.error('logger2 error message')
logger2.critical('logger2 critical message')
```

输出结果为

```text
2019-08-08 14:23:10,335 - root - DEBUG - logger1 debug message
2019-08-08 14:23:10,336 - root - INFO - logger1 info message
2019-08-08 14:23:10,336 - root - WARNING - logger1 warning message
2019-08-08 14:23:10,336 - root - ERROR - logger1 error message
2019-08-08 14:23:10,336 - root - CRITICAL - logger1 critical message
2019-08-08 14:23:10,337 - mylogger - INFO - logger2 info message
2019-08-08 14:23:10,337 - mylogger - INFO - logger2 info message
2019-08-08 14:23:10,337 - mylogger - WARNING - logger2 warning message
2019-08-08 14:23:10,337 - mylogger - WARNING - logger2 warning message
2019-08-08 14:23:10,337 - mylogger - ERROR - logger2 error message
2019-08-08 14:23:10,337 - mylogger - ERROR - logger2 error message
2019-08-08 14:23:10,337 - mylogger - CRITICAL - logger2 critical message
2019-08-08 14:23:10,337 - mylogger - CRITICAL - logger2 critical message
```

**对于一般的创建log实例的过程**：

可以看到 `getLogger()` 没有参数时，默认为root。

需要创建handler，对handler可以设置format，可以设置level等信息，最后把handler绑定到一个log实例。

常用的几个handler

```text
logging.StreamHandler 可以向类似与sys.stdout或者sys.stderr的任何文件对象(file object)输出信息
logging.FileHandler 用于向一个文件输出日志信息
logging.handlers.RotatingFileHandler 类似于上面的FileHandler，但是它可以管理文件大小。当文件达到一定大小之后，它会自动将当前日志文件改名，然后创建一个新的同名日志文件继续输出
logging.handlers.TimedRotatingFileHandler 和RotatingFileHandler类似，不过，它没有通过判断文件大小来决定何时重新创建日志文件，而是间隔一定时间就自动创建新的日志文件
```

另外还有向远程网络发送日志的handler，可见[官方文档](https://docs.python.org/2/library/logging.handlers.html#module-logging.handlers)

**多个log的行为**
可以看到mylogger每个语句打印了两次，因为每个mylogger在创建时实际上创建了 `root.mylogger` , 依次类推。
每个log的消息都会分发给所有祖先，每个祖先都会用handler处理消息。

log在同一个解释器内保证名字相同时使用同一个实例，这是显而易见的。

---------------

参考博客：
[https://blog.csdn.net/zyz511919766/article/details/25136485](https://blog.csdn.net/zyz511919766/article/details/25136485)
