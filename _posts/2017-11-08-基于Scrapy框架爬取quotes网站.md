---
layout:     post
title:  基于Scrapy框架爬取quotes网站
subtitle:   爬虫实例基于Scrapy框架
date:       2017-11-08
author:     QC
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Scrapy
---
### Scrapy框架爬取QuotesBot

#### 1.前言

今天这篇博客是使用Scrapy框架简单的爬取` http://quotes.toscrape.com` 网站的例子，刚开始学习Scrapy框架，如有不足，请多多指教。

#### 2.主要代码

items.py

```
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/items.html

import scrapy


class QuotesbotexItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass

```

middlewares.py

```
# -*- coding: utf-8 -*-

# Define here the models for your spider middleware
#
# See documentation in:
# http://doc.scrapy.org/en/latest/topics/spider-middleware.html

from scrapy import signals


class QuotesbotexSpiderMiddleware(object):
    # Not all methods need to be defined. If a method is not defined,
    # scrapy acts as if the spider middleware does not modify the
    # passed objects.

    @classmethod
    def from_crawler(cls, crawler):
        # This method is used by Scrapy to create your spiders.
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)
        return s

    def process_spider_input(self, response, spider):
        # Called for each response that goes through the spider
        # middleware and into the spider.

        # Should return None or raise an exception.
        return None

    def process_spider_output(self, response, result, spider):
        # Called with the results returned from the Spider, after
        # it has processed the response.

        # Must return an iterable of Request, dict or Item objects.
        for i in result:
            yield i

    def process_spider_exception(self, response, exception, spider):
        # Called when a spider or process_spider_input() method
        # (from other spider middleware) raises an exception.

        # Should return either None or an iterable of Response, dict
        # or Item objects.
        pass

    def process_start_requests(self, start_requests, spider):
        # Called with the start requests of the spider, and works
        # similarly to the process_spider_output() method, except
        # that it doesn’t have a response associated.

        # Must return only requests (not items).
        for r in start_requests:
            yield r

    def spider_opened(self, spider):
        spider.logger.info('Spider opened: %s' % spider.name)

```

pipelines.py


```
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: http://doc.scrapy.org/en/latest/topics/item-pipeline.html


class QuotesbotexPipeline(object):
    def process_item(self, item, spider):
        return item

```

toscrape-css.py
> 用CSS选择器

```
# -*- coding:utf-8 -*-
import scrapy

class ToScrapeCSSSpider(scrapy.Spider):
    name = "toscrape-css"
    start_urls=[
        'http://quotes.toscrape.com',
    ]

    def parse(self, response):
        for quote in response.css("div.quote"):
            yield {
                'text':quote.css("span.text::text").extract_first(),
                'author':quote.css("small.author::text").extract_first(),
                'tags':quote.css("div.tags > a.tags::text").extract()
            }
        next_page_url=response.css("li.next > a::attr(href)").extract_first()
        if next_page_url is not None:
            yield scrapy.Request(response.urljoin(next_page_url))
```

toscrape-xpath.py

```
# -*- coding:utf-8 -*-
import scrapy

class ToScrapeSpiderXPath(scrapy.Spider):
    name = 'toscrape-xpath'
    start_urls=[
        'http://quotes.toscrape.com/',
    ]
    def parse(self, response):
        for quote in response.xpath('//div[@class="quote"]'):
            yield {
                'text':quote.xpath('./span[@class="text"]/text()').extract_first(),
                'author':quote.xpath('.//small[@class="author"]/text()').extract_first(),
                'tags':quote.xpath('.//div[@class="tags"]/a[@class="tag"]/text()').extract()
            }
            next_page_url=response.xpath('//li[@class="next"]/a/@href').extract_first()
            if next_page_url is not None:
                yield scrapy.Request(response.urljoin(next_page_url))
```

#### 3.运行结果

##### 3.1运行 scrapy crawl toscrape-css -o quotes.json

![](https://i.imgur.com/I7hjrqe.png)

> 写出的json文件

![](https://i.imgur.com/rwaP5X7.png)

##### 3.2运行 scrapy crawl toscrape-xpath -o quotes1.json

![](https://i.imgur.com/JUajkJP.png)

> 写出的json文件

![](https://i.imgur.com/LFCtuUW.png)

> 摁钉 author by chongqin

