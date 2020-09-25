---
title: learn to asyncio async await aiohttp
description:
date: 2019-12-18
tags:

  + python

layout: layouts/post.njk
image:
---

* What's asyncio?

``` text
asyncio is a library to write concurrent code using the async/await syntax...
asyncio is used as a foundation for multiple Python asynchronous frameworks
that provide high-performance network and web-servers,
database connection libraries, distributed task queues, etc.
—— docs.python.org
`
```

## 引子

标题为实战体验，实际上是夹带私货、笔记性质的项目梳理，从 `scrapy` 框架中脱离首次使用 `asyncio` 的体验。博主偶然为一次资讯类项目采集数据，目标明确，风控反爬强悍。为什么不用 `scrapy` ? 因为具体业务简单，不需要这么重的框架， `scrapy` 的优势在这里也难以体现，要突破反爬和重点不在框架。正文通过伪代码的方式梳理项目。

异步操作相对于同步。Python工程师对于 `scrapy` 肯定不会陌生， `scrpay` 底层通过 `twisted` 来实现， `twisted` 本身是一个很优秀的异步网络框架，基于 `twisted` 的 `Server` 也有不少。在看 `scrapy` 源码和学习 `twisted` 过程中会看到很多 `defered` 、 `deferedList` 等延迟链。大量使用回调函数，工厂方法，初学者往往看得头大。

`asyncio` 很早就被提出，但 `API` 不是很好用，个人感觉没有 `Thread` 甚至不如 `future` 要容易理解，在 3.5 正式引入 `async` 和 `await` 语法糖后，越来越像 `node` 了...

## 项目梳理

首先将平时常用的包从 `scrapy` 项目中抽离，将 `proxy` 相关中间件抽出作为独立模块，便于以后引用。

按照项目的一般逻辑创建目录如下：

``` text

* xxx-crawler
  + xxx
    - __init__
      - settings
      - log.txt
      - xxx_spider
      - baseClass
      - run
      - http_proxy
          - __init__
          - proxy_utils
          - request_utils
  + requirements
  + build
  + ...

```

~~可以把 `http_utils` 包独立放在爬虫外面。个人喜好问题~~

 xxx_spider.py

``` python
# 伪代码
async def crawl:
    while cli.llen:
        url = cli.lpop
    async with ClientSession() as session:
        async with session.get(url, headers, proxy) as rep:
            data = await rep.text()
```

在第一次编写中出现了使用 `requests` 库导致异步效果很差的错误，异步网络库 `aiohttp` 的 `ClientSession` 基本上就够爬虫用了。

run.py

``` python
async def main():
    await [Asynchronous corotinue]
        await asyncio.gather(
            asyncio.create_task([Asynchronous corotinue]),
            asyncio.create_task([Asynchronous corotinue]),
            # an Asynchronous corotinue can be gather directly
            [Asynchronous corotinue]
)

def __init__ == __main__():
    asyncio.run(main())

```

`asyncio.run` 配合 `asyncio.create_task` 已经足够处理很多需要异步的逻辑。

## 疑问

用了异步效率变高了吗？

在我的项目中，因为爬取速率受限，分成6个协程，并在检测到被受限时， `asyncio.sleep()` ，在每次请求也加上 `sleep` 操作。程序是变异步了，但实际爬取速率只增加了一点，但没有出现我想要的协程穿插爬取的过程。这里需要一个优化异步网络的操作。

经过实践，在不到一分钟的时间内检测5个请求会关进小黑屋，具体是按照频率还是次数我们无法判断。一方面我们将每单个请求间隔延时10s，另一方面在请求失败时挂起这个协程，等待其他协程运行。速度总体提高了不少。

ps. 最近想攒点异步编程的知识。
