---
title: scrapy爬虫折腾系列-02
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
summary: crawlSpider爬虫
categories: python
tags: scrapy
keywords: scrapy
abbrlink: '95e45310'
date: 2019-09-16 13:08:52
---

# Scrapy折腾系列-02

## 1、笔记

1. response是一个`scrapy.http.response.html.HtmlResponse`对象，可执行`xpath`和`css`语法来提取数据
2. 提取出来的数据，是一个 `Selector`或者是一个`selectorList`对象，要想获取其中的字符串，得执行`getall`或者`get`方法
3. `getall`方法：获取`selector`中的所有文本，返回的是一个列表
4. `get`方法： 获取`selector`中的第一个文本，返回的是str
5. 如果数据解析回来，要传给pipeline执行，那么可以使用`yield`来返回，或者是在前面定义一个item集合，将后面收集到的item一个一个的append进去，最后在统一返回item集合
6. item: 建议在`items.py`文件中定义好模型，好比java中的bean,将它封装为一个对象，以后不要再使用字典，显得不专业
7. pipeline：这个是专门用来保存数据的，其中有3个方法会经常使用
   - `open_spider(self,spider)` :当爬虫被打开时执行
   - `process_item(slef,item,spider)`:当爬虫有items传过来时调用
   - `close_spider(self,spider)`:当爬虫关闭时调用
   - 要激活pipeline，应该在`setting.py`中，设置`item_pipeline`,将pipeline的注释取消掉
8. 在pipeline中需要使用json将字典转化为json,需要导入json,`import json`,所以需要把对象转化为字典：`dict(item)`

## 2、优化数据存储

刚才我们使用对象转化为字典，或者直接传入字典的方式方式，最后使用python自带的转化为json的库来转化数据并保存在文件中。

转化为json格式还有很多方法，比如这里使用的exporters的方法，先导入`from scrapy.exporters import  JsonLinesItemExporter` 或者JsonItemExporter

## 3、抓取多个页面

1. 在parse方法体里定义一个base_domain字符串，你要采集的域名
2. 在获取单个页面内容的下面判断是否有下一个页面存在
   1. 如果没有，return出去，如果有，就调用`scrapy.Request()`方法，Request方法有2个参数，一个是url,另外一个是callback回调函数，在这里，我们先将url拼接好，然后回调函数就是本身的parse函数，即`callback=self.parse`,回调函数不写`（）`,最后再yield出去就行了，yield表示的是返回当前的这一个请求，而return是返回当前请求并跳出控制语句

## 4、CrawlSpider

满足定义规则的url就会执行，去调用crawlSpider，不用像传统的爬虫一样，需要手动调用yield scrapy.Request()

创建爬虫: `scrapy genspider -t crawl  爬虫名 域名`

查看网页<http://www.wxapp-union.com/portal.php?mod=list&catid=2>，查找其中的规律：

文章详情页：　http://www.wxapp-union.com/article-5536-1.html>　　根据-5536-1来表示文章

教程页：　 http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1　根据ｐａｇｅ变化来翻页

1. 编写start_urls(初始爬取页面): start_urls = ['http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1'] 从第一页爬取

2. 编写rule规则：  如果只是想要获取url列表，不需要回调函数，如果是需要返回文章响应的数据，就需要回调函数，接下来定义rule规则：

   ```
   rules = (
           Rule(LinkExtractor(allow=r'http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=\d'), follow=True),
           Rule(LinkExtractor(allow=r'http://www.wxapp-union.com/article-.+\.html'), callback="parse_item", follow=True),
       )
   ```

3. 重写parse_item

   ```
       def parse_item(self, response):
           title = response.xpath('//div[@class="cl"]/h1/text()').get()
           print(title)
   ```

   先打印title看看：

4. 配置settting.py

   ```
   1.请求头
   # Override the default request headers:
   DEFAULT_REQUEST_HEADERS = {
     'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
     'Accept-Language': 'en',
       'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.19 Safari/537.36'
   }
   2.机器人规则
   # Obey robots.txt rules
   ROBOTSTXT_OBEY = False
   3.设置下载速度，防止被封ip
   DOWNLOAD_DELAY = 1
   ```

   5.cmd命令太麻烦，自己编写py文件执行

   ```
   #调用scrapy的cmdline模块，来实现命令行操作
   # -*- coding: utf-8 -*-
   from scrapy import  cmdline

   cmdline.execute("scrapy crawl wxapppp_spider".split())
   ```

   6.运行

   ```
   2019-09-16 11:20:34 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5525-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   微信小程序之onLaunch与onload异步问题 
   2019-09-16 11:20:35 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5526-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   烧脑！JS+Canvas带你体验「偶消奇不消」的智商挑战 
   2019-09-16 11:20:37 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5527-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   微信小程序使用多色图标详解 
   2019-09-16 11:20:38 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5528-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   微信小程序 select 下拉框组件 
   2019-09-16 11:20:39 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5529-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   基于小程序·云开发构建高考查分小程序丨实战 
   2019-09-16 11:20:41 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5530-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   利用微信电脑最新版 反编译微信小程序 无需root 
   2019-09-16 11:20:42 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5534-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   微信小程序路由栈不能超过 10 的解决方案 
   2019-09-16 11:20:43 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5535-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   微信小程序自定义头部返回按钮及回到首页样式 
   2019-09-16 11:20:45 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5533-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   小程序底层实现原理及一些思考 
   2019-09-16 11:20:46 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5536-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   用uniapp写个天气的微信小程序和支付宝小程序 
   2019-09-16 11:20:47 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5537-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   微信小程序上传照片后旋转问题解决 
   2019-09-16 11:20:49 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5540-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   微信小程序实现pdf,word等格式文件上传 
   2019-09-16 11:20:50 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5538-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   Taro编写微信小程序时，自定义组件样式引入后不生效 
   ```

   7.接下来获取作者 时间

   ```
   //p[@class="authors"]/a/text()   xpath表达式  作者
   //p[@class="authors"]/span/text()   时间
   ```

   ```python
       def parse_item(self, response):
           title = response.xpath('//div[@class="cl"]/h1/text()').get()
           # print(title)
           author = response.xpath('//p[@class="authors"]/a/text()').get()
           time = response.xpath('//p[@class="authors"]/span/text()').get()

           print("author: %s/pub_time: %s/title: %s"%(author,time,title))
           print('*'*30)

   ```

   ```
   运行start.py
   运行结果:
   author: Rolan/pub_time: 2019-8-26 00:09/title: 用小程序·云开发两天搭建mini论坛丨实战 
   ******************************
   2019-09-16 11:41:44 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5477-1.html> (referer: http://www.wxapp-union.com/article-5475-1.html)
   author: Rolan/pub_time: 2019-8-19 00:47/title: 微信小程序canvas绘制圆角base64图片 
   ******************************
   2019-09-16 11:41:45 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5486-1.html> (referer: http://www.wxapp-union.com/article-5475-1.html)
   author: Rolan/pub_time: 2019-8-21 00:52/title: 小程序自定义弹层禁止页面滚动方案详解 
   ******************************
   2019-09-16 11:41:46 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5478-1.html> (referer: http://www.wxapp-union.com/article-5475-1.html)
   author: Rolan/pub_time: 2019-8-19 00:51/title: 扫小程序码实现网站登陆,提供源代码 
   ******************************
   2019-09-16 11:41:48 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5476-1.html> (referer: http://www.wxapp-union.com/article-5475-1.html)
   author: Rolan/pub_time: 2019-8-19 00:37/title: uni-app微信小程序接入人脸核身SDK 
   ******************************
   2019-09-16 11:41:49 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5462-1.html> (referer: http://www.wxapp-union.com/article-5475-1.html)
   author: Rolan/pub_time: 2019-8-14 00:49/title: 微信小程序 - 输入起点、终点获取距离并且进行路线规划（腾讯地图） ... ... 
   ******************************
   2019-09-16 11:41:51 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-4993-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   author: Rolan/pub_time: 2019-3-18 00:32/title: 微信小程序开发中遇到的问题及解决办法（一） 
   ******************************
   2019-09-16 11:41:53 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-4997-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   author: Rolan/pub_time: 2019-3-19 10:16/title: 微信小程序开发之多图片上传+服务端接收 
   ******************************
   2019-09-16 11:41:53 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5531-1.html> (referer: http://www.wxapp-union.com/article-5540-1.html)
   author: Rolan/pub_time: 2019-9-11 00:01/title: 小程序上线多项新能力，“服务商助手”C位登场 
   ******************************
   2019-09-16 11:41:54 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5522-1.html> (referer: http://www.wxapp-union.com/article-5540-1.html)
   author: Rolan/pub_time: 2019-9-4 14:12/title: 这些小程序技巧，你敢说你一个用不到？ 
   ******************************
   2019-09-16 11:41:55 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5011-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   author: Rolan/pub_time: 2019-3-22 00:34/title: 微信小程序开发中的代码片段总结 
   ******************************
   2019-09-16 11:41:56 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5056-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   author: Rolan/pub_time: 2019-4-4 00:24/title: 微信小程序开发早知道 
   ******************************
   2019-09-16 11:41:57 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-4794-1.html> (referer: http://www.wxapp-union.com/article-4793-1.html)
   author: Rolan/pub_time: 2019-1-2 14:48/title: 一诞小程序总结 
   ******************************
   2019-09-16 11:41:58 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-4792-1.html> (referer: http://www.wxapp-union.com/article-4793-1.html)
   author: Rolan/pub_time: 2019-1-2 00:12/title: 你的年目标实现了吗，记一次开发微信小程序 
   ******************************
   2019-09-16 11:41:59 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5510-1.html> (referer: http://www.wxapp-union.com/article-5511-1.html)
   author: Rolan/pub_time: 2019-8-30 00:08/title: 微信小程序生成自适应海报 
   ******************************
   2019-09-16 11:42:01 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5482-1.html> (referer: http://www.wxapp-union.com/article-5480-1.html)
   author: Rolan/pub_time: 2019-8-21 00:07/title: 微信小程序 webview 与 h5 实时通讯的实现 
   ******************************
   2019-09-16 11:42:02 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5483-1.html> (referer: http://www.wxapp-union.com/article-5484-1.html)
   author: Rolan/pub_time: 2019-8-21 00:15/title: 解决小程序中webview页面多层history返回问题 
   ******************************
   2019-09-16 11:42:04 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5499-1.html> (referer: http://www.wxapp-union.com/article-5496-1.html)
   author: Rolan/pub_time: 2019-8-27 00:09/title: 微信小程序和Jenkins不得不说的二三事 
   ******************************
   2019-09-16 11:42:04 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5495-1.html> (referer: http://www.wxapp-union.com/article-5496-1.html)
   author: Rolan/pub_time: 2019-8-26 00:06/title: 【微信小程序】图片压缩-纯质量压缩，非长宽裁剪压缩 
   ******************************
   2019-09-16 11:42:06 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5494-1.html> (referer: http://www.wxapp-union.com/article-5486-1.html)
   author: Rolan/pub_time: 2019-8-26 00:02/title: 研究了多个出行类小程序后，我们发现了留住用户的秘密 
   ******************************
   2019-09-16 11:42:07 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5112-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   author: Rolan/pub_time: 2019-4-23 00:43/title: 微信小程序开发——点击按钮退出小程序的实现 
   ******************************
   2019-09-16 11:42:07 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5137-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   author: Rolan/pub_time: 2019-5-5 00:42/title: 微信小程序开发需要注意的一些规范 
   ******************************
   ```

   8.获取内容

   ```
           content = response.xpath('//td[@id="article_content"]//text()').getall()

           # print("author: %s/pub_time: %s/title: %s"%(author,time,title))
           print(content)
           print('*'*30)
   ```

   显示：

   ```
   2019-09-16 12:32:20 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5536-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   [' \n                     \n                    ', '经过最近两年多的发展，', '小程序', '的地位也逐渐越来越高了，各个平台前赴后继做了自家的小程序平台，随着市场的需求越来愈多，我们开发各平台的小程序的激情也随（被）之（逼）高（无）涨（奈）。', '\r\n', '为何选择uniapp？uni-app 是一个使用 Vue.js 开发所有前端应用的框架，开发者编写一套代码，可发布到iOS、Android、H5、以及各种小程序（微信/支付宝/百度/头条/QQ/钉钉）等多个平台。即使不跨端，uni-app同时也是更好的小程序开发框架。来自官方。喜欢taro， wepy，mpvue的朋友也莫喷我，大家各有所好，大家开心就好。', '\r\n', '智行天气小程序（支付宝小程序、微信小程序）', '\r\n', '\r\n', '效果图', '\r\n', '1、获取位置信息', '\r\n', '在定位功能中，本程序用到腾讯地图的api 以及 腾讯天气的api接口，', '\r\n', '\r\n', '需要到官网中注册开发者账号，通过注册后得到的appKey来请求我们需要的数据，详细注册步骤请自行度娘', '\r\n', '\r\n', '由于需要用到定位功能，uniapp的getLocation方法获取到的是当前位置的坐标，然后对应百度地图具体城市', '\r\n', ......
   ```

   在这里，他会显示我门这是一个list,不能用str类型输出，所以我们要把list转化为str

   ```
           article_content = "".join(content).strip() #转化为字符串类型并去除空白
   ```

   ```
   2019-09-16 12:36:55 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5475-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
   [' \n                     \n                    ', '今天下午突然听到群里有人说微信', '小程序', '工具更新了,文档也更新了不少内容.', '顾不上吃冬至的饺子.我就冲进来了.', '先说分享功能,目前真机尚不能调试.开发工具上可以看看效果.后续还会更新.', 'Page()中加上如下代码后在右上角就会出现三个小白点', 'title:分享的标题.', 'desc:分享一段描述.', 'path:这个参数有点意思.以前在微信中的分享一般都是url.这里是当前页面这里应该是pages/index?id=123这里的id目前还不知道是什么.', '也就是说以后你可以在微信中像分享一个网页一样分享一个页面了.', 'onShareAppMessage: ', 'function', ' ', '()', ' {\r\n    ', 'return', ' {\r\n      title: ', "'垃圾分类黑板报'", ',\r\n      desc: ', "'垃圾分类就选垃圾分类黑板报!'", ',\r\n      path: ', "'/page/user?id=123'", '\r\n    }\r\n  }', '分享参数用处:', '我这里没有用到路径后的参数,说个场景:参数是用户昵称,A分享了XXX小程序到微信群里,B点开小程序,弹个toast,”来自A的分享”.', ' ']
   ```



## 5、存json入文件

1、封装对象  --->items.py

```
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# https://docs.scrapy.org/en/latest/topics/items.html

import scrapy


class WxAppItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    # pass
    title = scrapy.Field()
    author =scrapy.Field()
    time =scrapy.Field()
    content =scrapy.Field()

```

2.把数据保存在json文件中 --->pipeline.py

因为content内容很多，所以我们使用JsonLinesItemExporter类来处理，先映入

```
from scrapy.exporters import JsonLinesItemExporter
```

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://docs.scrapy.org/en/latest/topics/item-pipeline.html

from scrapy.exporters import JsonLinesItemExporter

class WxAppPipeline(object):
    def __init__(self):
        self.fp = open('wxapp.json','wb')
        # 使用jsonLinesExporter
        self.exporter = JsonLinesItemExporter(self.fp,ensure_ascii=False,encoding="utf-8")

    # 写入文件
    def process_item(self, item, spider):
        self.exporter.export_item(item)
        return item
    
    # 关闭文件
    def close_spider(self,spider):
        self.fp.close()
```

3.返回数数item

```
    def parse_item(self, response):
        title = response.xpath('//div[@class="cl"]/h1/text()').get()
        # print(title)
        author = response.xpath('//p[@class="authors"]/a/text()').get()
        time = response.xpath('//p[@class="authors"]/span/text()').get()
        content = response.xpath('//td[@id="article_content"]//text()').getall()
        article_content = "".join(content).strip() #转化为字符串类型并去除空白
        item = WxAppItem(title=title, author=author, time=time, content=content)
        yield  item
```

将数据item   `yeild`出去

运行start.py

发现在运行了：

```
 'time': '2019-9-9 00:23',
 'title': '微信小程序 select 下拉框组件 '}
2019-09-16 12:52:02 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.wxapp-union.com/article-5529-1.html> (referer: http://www.wxapp-union.com/portal.php?mod=list&catid=2&page=1)
2019-09-16 12:52:02 [scrapy.core.scraper] DEBUG: Scraped from <200 http://www.wxapp-union.com/article-5529-1.html>
{'author': 'Rolan',
 'content': [' \n 
```

但是问题来了，我们不是要写入wxapp.json文件吗，为什么项目中没有生成该文件呢？

原因是：我们没有打开pipeline

解决方案：

```
ITEM_PIPELINES = {
   'wx_app.pipelines.WxAppPipeline': 300,
}
```

在运行一次：就发现有该文件了打开wxapp.json

```
{"title": "微信小程序开发之页面分享 onShareAppMessage 分享参数用处 ", "author": "Rolan", "time": "2019-8-19 00:08", "content":[.....]}
```

## 6、总结

### 6.1crawlspider

需要使用`linkExtractor`和`rule`，这两个东西决定爬虫的具体走向

1.allow设置规则：要能够限制在我们指定的想要的url，不要与其他url产生相同的则正表达式。

2.什么时候使用follow： 再爬取时，需要将满足当前条件的url进一步跟进时，就设置为true,否则设置为false

3.什么时候该指定callback: 如果这个url对应的页面只是为了获取更多的url,并不需要里面的数据，那么可以不指定callback