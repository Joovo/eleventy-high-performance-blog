---
title: base extensions in scrapy
description:
date: 2020-01-13
tags:
  - python
layout: layouts/post.njk
image:
---

``` python
EXTENSIONS_BASE = {
    'scrapy.extensions.corestats.CoreStats': 0,
    'scrapy.extensions.telnet.TelnetConsole': 0,
    'scrapy.extensions.memusage.MemoryUsage': 0,
    'scrapy.extensions.memdebug.MemoryDebugger': 0,
    'scrapy.extensions.closespider.CloseSpider': 0,
    'scrapy.extensions.feedexport.FeedExporter': 0,
    'scrapy.extensions.logstats.LogStats': 0,
    'scrapy.extensions.spiderstate.SpiderState': 0,
    'scrapy.extensions.throttle.AutoThrottle': 0,
}
```

## scrapy.extensions.corestats. CoreStats

``` python
class CoreStats(object):
    # stats 对应 scrapy.StatsCollector.statscollectors
    stats 内维护一个 dict 和几个 get/set 方法
    def __init__(self, stats):
        self.stats = stats

    @classmethod
    def from_crawler(cls, crawler):
        o = cls(crawler.stats)
        # 连接几个signals
        crawler.signals.connect(o.spider_opened, signal=signals.spider_opened)
        crawler.signals.connect(o.spider_closed, signal=signals.spider_closed)
        crawler.signals.connect(o.item_scraped, signal=signals.item_scraped)
        crawler.signals.connect(o.item_dropped, signal=signals.item_dropped)
        crawler.signals.connect(o.response_received, signal=signals.response_received)
        return o

    def spider_opened(self, spider):
        # 设置开启时间
        self.stats.set_value('start_time', datetime.datetime.utcnow(), spider=spider)

    def spider_closed(self, spider, reason):
        finish_time = datetime.datetime.utcnow()
        elapsed_time = finish_time - self.stats.get_value('start_time')
        elapsed_time_seconds = elapsed_time.total_seconds()
        self.stats.set_value('elapsed_time_seconds', elapsed_time_seconds, spider=spider)
        self.stats.set_value('finish_time', finish_time, spider=spider)
        self.stats.set_value('finish_reason', reason, spider=spider)

    def item_scraped(self, item, spider):
        # 计数
        self.stats.inc_value('item_scraped_count', spider=spider)

    def response_received(self, spider):
        self.stats.inc_value('response_received_count', spider=spider)

    def item_dropped(self, item, spider, exception):
        reason = exception.__class__.__name__
        self.stats.inc_value('item_dropped_count', spider=spider)
        self.stats.inc_value('item_dropped_reasons_count/%s' % reason, spider=spider)
```
