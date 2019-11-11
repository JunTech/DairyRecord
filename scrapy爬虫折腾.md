# scrapy爬虫折腾

## 1、scrapy爬虫入门

1. scrapy是框架，好比一辆车子，beautifulsoup好比一个轮子，所以我们只要会开车就行了
2. 采用的是异步框架   基于Twisted，实现高效率的网络采集
3. 最强大的爬虫框架 

## 2、scrapy安装

1. ### pip install scrapy

   *可能会出现错误-->vc++14.0 Twisted

   ​	解决方法：离线安装pip install xxx.whl;

   *scrapy bench运行时报错  --->pywin32

   ​	解决方法： pip install pywin32

   或者参考我的另外一篇文章： 把pip换成清华源，这样就可以为所欲为啦：

   传送门： [python清华源安装scrapy](https://juntech.top/posts/5ea02dcd.html)

## 3、scrapy原理

1. ​	爬虫文件 发送  请求/网络链接
2. ​        引擎发送请求/网址给调度器
3. ​        调度器将请求/网址入队
4. ​        调度器将入队的请求返回给引擎
5. ​        引擎将入队后返回的请求发送给下载器
6. ​        下载器得到互联网的响应返回给引擎
7. ​         引擎将响应发送给爬虫文件
8. ​        爬虫文件读取数据，发出新的请求
9. ​        引擎进行请求的判断，是继续请求还是将数据发送到管道文件（数据库或者txt)保存



## 4、scrapy入门

要想知道scrapy有哪些方法，在cmd里输入scrapy，他下面就会列出一大推scrapy的方法，下面我们就来使用

### 4.1新建项目

scrapy startproject xxx(项目名称)

```shell
scrapy startproject 项目名称，这里我们打算爬取西刺代理  xicidailiSpider
```

### 4.2 创建爬虫

scrapy genspider 爬虫名称 域名

```
scrapy genspider xicidaili(爬虫名称) xicidaili.com
```

注意：

​	*上面是项目名称，下面是爬虫名称，不要弄成一样的了。

​	*网站域名是允许爬虫采集的域名

打开项目，我们就可以发现spider文件夹下有xicidaili.py文件了，这就是我们的爬虫文件！！！

### 4.3 Robots协议

```
# Obey robots.txt rules
ROBOTSTXT_OBEY = True
```

将True改为False： 不然爬虫没法工作

到网站根目录，在后面添加后缀 robots.txt,就可以看到

### 4.4 爬虫文件介绍

```python
# -*- coding: utf-8 -*-
import scrapy   #导入scrapy包

#创建爬虫类
class XicidailiSpider(scrapy.Spider):  #继承自scrapy.Spider类
    name = 'xicidaili'   #爬虫名
    allowed_domains = ['xicidaili.com'] #允许爬取的域名
    start_urls = ['http://xicidaili.com/'] #开始采集的网址

#解析响应数据，提取数据获取网址等  response就是网页源码
    def parse(self, response):
        pass
```

### 4.5分析网站

* 提取数据   
  * 正则（基础  必回  ）
  * xpath -------->从html中提取数据语法
    * response.xpath('xpath表达式').get()  获取一个数据
    * response.xpath('xpath表达式').getall()  获取多个数据
    * response.xpath('xpath表达式').extract_first()  和get同样的效果
  * css

### 4.6提取数据

```python
# -*- coding: utf-8 -*-
import scrapy   #导入scrapy包

#创建爬虫类
class XicidailiSpider(scrapy.Spider):  #继承自scrapy.Spider类
    name = 'xicidaili'   #爬虫名
    allowed_domains = ['xicidaili.com'] #允许爬取的域名
    # start_urls = ['http://xicidaili.com/'] #开始采集的网址
    start_urls = ['https://www.xicidaili.com/nn/']

#解析响应数据，提取数据获取网址等  response就是网页源码
    def parse(self, response):
        # pass   方法
        #提取数据
        # response.xpath('表达式')
        #提取ip  port
        # response.xpath('//tr//td[2]/text()')  #ip
        # response.xpath('//tr//td[3]/text()')  #port
        # 但我们不这么写
        #1、因为全在tr标签下，拿到tr标签的内容在选择
        selectors = response.xpath('//tr')   #选择所有的tr标签
        for selector in selectors:  #遍历tr标签下的所有的td标签
            ip = selector.xpath('./td[2]/text()')  #   ./ 表示在当前节点下继续选择
            port = selector.xpath('./td[3]/text()') #在当前节点下继续选择

            print(ip,port)  #打印ip  port
```

### 4.7运行爬虫

```python
scrapy crawl （爬虫名）xicidaili
```

你满心欢喜的会发现报错了：

```json
2019-09-15 20:22:36 [scrapy.utils.log] INFO: Scrapy 1.7.2 started (bot: xicidailiSpider)
2019-09-15 20:22:36 [scrapy.utils.log] INFO: Versions: lxml 4.3.4.0, libxml2 2.9.5, cssselect 1.0.3, parsel 1.5.1, w3lib 1.20.0, Twisted 19.2.1, Python 3.7.0 (v3.7.0:1bf9cc5093, Jun 27 2018, 04:59:51) [MSC v.1914 64 bit (AMD64)], pyOpenSSL 19.0.0 (OpenSSL 1.1.1c  28 May 2019), cryptography 2.7, Platform Windows-10-10.0.17134-SP0
2019-09-15 20:22:36 [scrapy.crawler] INFO: Overridden settings: {'BOT_NAME': 'xicidailiSpider', 'NEWSPIDER_MODULE': 'xicidailiSpider.spiders', 'SPIDER_MODULES': ['xicidailiSpider.spiders']}
2019-09-15 20:22:36 [scrapy.extensions.telnet] INFO: Telnet Password: 1f33d178e985a6c0
2019-09-15 20:22:36 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.corestats.CoreStats',
 'scrapy.extensions.telnet.TelnetConsole',
 'scrapy.extensions.logstats.LogStats']
2019-09-15 20:22:36 [scrapy.middleware] INFO: Enabled downloader middlewares:
['scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
 'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware',
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2019-09-15 20:22:36 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2019-09-15 20:22:36 [scrapy.middleware] INFO: Enabled item pipelines:
[]
2019-09-15 20:22:36 [scrapy.core.engine] INFO: Spider opened
2019-09-15 20:22:36 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2019-09-15 20:22:36 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023
2019-09-15 20:22:37 [scrapy.downloadermiddlewares.retry] DEBUG: Retrying <GET https://www.xicidaili.com/nn/> (failed 1 times): 503 Service Unavailable
2019-09-15 20:22:37 [scrapy.downloadermiddlewares.retry] DEBUG: Retrying <GET https://www.xicidaili.com/nn/> (failed 2 times): 503 Service Unavailable
2019-09-15 20:22:37 [scrapy.downloadermiddlewares.retry] DEBUG: Gave up retrying <GET https://www.xicidaili.com/nn/> (failed 3 times): 503 Service Unavailable
2019-09-15 20:22:37 [scrapy.core.engine] DEBUG: Crawled (503) <GET https://www.xicidaili.com/nn/> (referer: None)
2019-09-15 20:22:37 [scrapy.spidermiddlewares.httperror] INFO: Ignoring response <503 https://www.xicidaili.com/nn/>: HTTP status code is not handled or not allowed
2019-09-15 20:22:37 [scrapy.core.engine] INFO: Closing spider (finished)
2019-09-15 20:22:37 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 657,
 'downloader/request_count': 3,
 'downloader/request_method_count/GET': 3,
 'downloader/response_bytes': 999,
 'downloader/response_count': 3,
 'downloader/response_status_count/503': 3,
 'elapsed_time_seconds': 0.60528,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2019, 9, 15, 12, 22, 37, 561550),
 'httperror/response_ignored_count': 1,
 'httperror/response_ignored_status_count/503': 1,
 'log_count/DEBUG': 4,
 'log_count/INFO': 11,
 'response_received_count': 1,
 'retry/count': 2,
 'retry/max_reached': 1,
 'retry/reason_count/503 Service Unavailable': 2,
 'scheduler/dequeued': 3,
 'scheduler/dequeued/memory': 3,
 'scheduler/enqueued': 3,
 'scheduler/enqueued/memory': 3,
 'start_time': datetime.datetime(2019, 9, 15, 12, 22, 36, 956270)}
2019-09-15 20:22:37 [scrapy.core.engine] INFO: Spider closed (finished)

```

报的啥错呢？一看

```
downloader/response_status_count/503': 3, 
```

503错误，服务器端的，究竟是怎么回事呢？

**补充：常见可能被网站识别返回错误**
1、CAPTCHApages （captcha，验证码）

2、Unusualcontent delivery delay  （响应时间、速度变慢了）

3、Frequentresponse with HTTP404,301 or 50x errors

（1）301 MovedTemporarily

（2）401unauthorized

（3）403forbidden  （aAatch处理的）

（4）404 notfound

（5）408 requesttimeout

（6）429 too manyrequests

（7）503 serviceunavailable  （ip层）

### 4.8原因分析：

1、在settings中将rebots改为False，发现依旧返回503错误

```
Obey robots.txt rules

ROBOTSTXT_OBEY = False
```

2、再分析，浏览器中打开正常显示，scrapy跟浏览器请求是同一个ip，所以排除ip封锁问题。

     网页不需要登录，排除cookie问题，剩下就是headers中user_agent的问题了。scrapy中添加user_agent之后，成功解决

     结论：没有请求头，被网站识别为爬虫程序（所以要模拟浏览器访问）
所以我们要给他添加请求头header

### 4.9虫振雄风

按照4.8所讲，我们需要添加请求头，那么添加在哪呢？

在setting.py文件里找到如下代码：

```
# Override the default request headers:
DEFAULT_REQUEST_HEADERS = {
  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
  'Accept-Language': 'en',
  'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1'
}

```

将其取消注释并添加请求头，运行：

```
/text()' data='9999'>]
[<Selector xpath='./td[2]/text()' data='120.83.103.135'>] [<Selector xpath='./td[3]/text()' data='9999'>]
[<Selector xpath='./td[2]/text()' data='120.83.122.28'>] [<Selector xpath='./td[3]/text()' data='9999'>]
[<Selector xpath='./td[2]/text()' data='14.115.206.93'>] [<Selector xpath='./td[3]/text()' data='61234'>]
[<Selector xpath='./td[2]/text()' data='125.123.122.206'>] [<Selector xpath='./td[3]/text()' data='9999'>]


```

但是有一个问题，我们要的数据保存在一个对象里面，我们要把他取出来：

### 4.10数据整理

我们使用get/extract_first/getall方法来获取数据对象里的数据，在这里get/extract_firs所取得的效果是一样的，随意使用，接下来我们继续完善我们的爬虫文件：

```python
# -*- coding: utf-8 -*-
import scrapy   #导入scrapy包

#创建爬虫类
class XicidailiSpider(scrapy.Spider):  #继承自scrapy.Spider类
    name = 'xicidaili'   #爬虫名
    allowed_domains = ['xicidaili.com'] #允许爬取的域名
    # start_urls = ['http://xicidaili.com/'] #开始采集的网址
    start_urls = ['https://www.xicidaili.com/nn/']

#解析响应数据，提取数据获取网址等  response就是网页源码
    def parse(self, response):
        # pass   方法
        #提取数据
        # response.xpath('表达式')
        #提取ip  port
        # response.xpath('//tr//td[2]/text()')  #ip
        # response.xpath('//tr//td[3]/text()')  #port
        # 但我们不这么写
        #1、因为全在tr标签下，拿到tr标签的内容在选择
        selectors = response.xpath('//tr')   #选择所有的tr标签
        for selector in selectors:  #遍历tr标签下的所有的td标签
            ip = selector.xpath('./td[2]/text()').get()  #   ./ 表示在当前节点下继续选择
            port = selector.xpath('./td[3]/text()').get() #在当前节点下继续选择
			
            #ip = selector.xpath('./td[2]/text()').extract_first()  #  ./ 表示在当前节点下继续选择
            #port = selector.xpath('./td[3]/text()').extract_first() #在当前节点下继续选择
            print(ip,port)  #打印ip  port
```

运行爬虫：

```
scrapy crawl xicidaili
```

运行结果：

```
112.87.71.119 9999
144.123.68.159 9999
110.86.136.82 9999
110.86.136.128 9999
171.11.178.251 9999
120.83.105.156 9999
171.35.149.199 9999
125.125.131.41 9999
120.83.102.74 9999
1.197.203.187 9999
120.84.100.140 808
115.207.253.27 9999
120.83.101.77 9999
120.83.102.181 9999
27.43.185.13 9999
1.197.11.20 9999
120.83.103.135 9999
120.83.122.28 9999
14.115.206.93 61234
125.123.122.206 9999

```

这样就拿到了我们想要的东西

### 4.11 问题来了

我们这里只获取到了一个页面的数据，但现实中我们要爬取的远不止一个页面的，拿xicidaili来说，他共有3823页，我们从最初级的开始 使用for循环，遍历看看：

遍历所有页面：

```
start_urls = ['https://www.xicidaili.com/nn/{page}' for page in range(1,3823)]
```

问题又来了，假如以后该网站继续添加页面数，那怎么办？

我们发现在最后一页的next_page为disable,为空，所以就好办了。

我们在第一页时使用xpath获取下一页的链接地址：(在这里推荐一款插件：xpath helper)

```xpath
//a[@class="next_page"]/@href
```

显示：

```
/nn/2
```

爬虫文件：

```
# -*- coding: utf-8 -*-
import scrapy   #导入scrapy包

#创建爬虫类
class XicidailiSpider(scrapy.Spider):  #继承自scrapy.Spider类
    name = 'xicidaili'   #爬虫名
    allowed_domains = ['xicidaili.com'] #允许爬取的域名
    # start_urls = ['http://xicidaili.com/'] #开始采集的网址
    start_urls = ['https://www.xicidaili.com/nn/5']
    # start_urls = ['https://www.xicidaili.com/nn/{page}' for page in range(1,3823)]



#解析响应数据，提取数据获取网址等  response就是网页源码
    def parse(self, response):
        # pass   方法
        #提取数据
        # response.xpath('表达式')
        #提取ip  port
        # response.xpath('//tr//td[2]/text()')  #ip
        # response.xpath('//tr//td[3]/text()')  #port
        # 但我们不这么写
        #1、因为全在tr标签下，拿到tr标签的内容在选择
        selectors = response.xpath('//tr')   #选择所有的tr标签
        for selector in selectors:  #遍历tr标签下的所有的td标签
            ip = selector.xpath('./td[2]/text()').get()  #   ./ 表示在当前节点下继续选择
            port = selector.xpath('./td[3]/text()').get() #在当前节点下继续选择

            print(ip,port)  #打印ip  port

        #翻页操作
        next_page = response.xpath('//a[@class="next_page"]/@href').get()
        if next_page:
            print(next_page)

```

运行：

```
222.89.32.159 9999
118.187.58.34 53281
112.85.170.219 9999
1.197.203.67 9999
163.204.243.0 9999
222.89.32.135 9999
163.204.242.93 9999
163.204.244.149 9999
49.70.32.68 9999
123.163.96.221 9999
163.204.245.243 9999
112.85.171.232 9999
58.253.157.130 9999
182.34.32.238 9999
1.199.31.244 9999
120.83.96.109 9999
120.83.101.212 9999
/nn/6


```



新的爬虫文件：

```
# -*- coding: utf-8 -*-
import scrapy   #导入scrapy包

#创建爬虫类
class XicidailiSpider(scrapy.Spider):  #继承自scrapy.Spider类
    name = 'xicidaili'   #爬虫名
    allowed_domains = ['xicidaili.com'] #允许爬取的域名
    # start_urls = ['http://xicidaili.com/'] #开始采集的网址
    start_urls = ['https://www.xicidaili.com/nn/5']
    # start_urls = ['https://www.xicidaili.com/nn/{page}' for page in range(1,3823)]



#解析响应数据，提取数据获取网址等  response就是网页源码
    def parse(self, response):
        # pass   方法
        #提取数据
        # response.xpath('表达式')
        #提取ip  port
        # response.xpath('//tr//td[2]/text()')  #ip
        # response.xpath('//tr//td[3]/text()')  #port
        # 但我们不这么写
        #1、因为全在tr标签下，拿到tr标签的内容在选择
        selectors = response.xpath('//tr')   #选择所有的tr标签
        for selector in selectors:  #遍历tr标签下的所有的td标签
            ip = selector.xpath('./td[2]/text()').get()  #   ./ 表示在当前节点下继续选择
            port = selector.xpath('./td[3]/text()').get() #在当前节点下继续选择

            print(ip,port)  #打印ip  port

        #翻页操作
        next_page = response.xpath('//a[@class="next_page"]/@href').get()
        if next_page:
            print(next_page)
            #拼接网址
            # next_url = "https://www.xicidaili.com"+next_page;
            next_url = response.urljoin(next_page)
            print(next_url)
            #发出请求   Request callback是回调函数，将请求得到响应交给自己处理
            yield scrapy.Request(next_url,callback=self.parse) #生成器
```

 scrapy.Request(next_url,callable=self.parse) #生成器

Request()发出请求，类似于requests.get()

callback 是将发出去的请求得到的响应交还给自己处理

注意： 回调函数不要写（），只写函数名

### 4.12执行程序，输出保存格式

```
scrapy crawl xicidaili -o ip.json  (json文件) /ip.csv(csv文件)
```

输出一下内容：

```
2019-09-15 22:10:39 [scrapy.extensions.feedexport] INFO: Stored csv feed (59 items) in: ip.csv
2019-09-15 22:10:39 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 37888,
 'downloader/request_count': 62,
 'downloader/request_method_count/GET': 62,
 'downloader/response_bytes': 412745,
 'downloader/response_count': 62,
 'downloader/response_status_count/200': 59,
 'downloader/response_status_count/503': 3,
 'elapsed_time_seconds': 10.825807,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2019, 9, 15, 14, 10, 39, 417655),
 'httperror/response_ignored_count': 1,
 'httperror/response_ignored_status_count/503': 1,
 'item_scraped_count': 59,
 'log_count/DEBUG': 122,
 'log_count/INFO': 12,
 'request_depth_max': 59,
 'response_received_count': 60,
 'retry/count': 2,
 'retry/max_reached': 1,
 'retry/reason_count/503 Service Unavailable': 2,
 'scheduler/dequeued': 62,
 'scheduler/dequeued/memory': 62,
 'scheduler/enqueued': 62,
 'scheduler/enqueued/memory': 62,
 'start_time': datetime.datetime(2019, 9, 15, 14, 10, 28, 591848)}
2019-09-15 22:10:39 [scrapy.core.engine] INFO: Spider closed (finished)
/nn/6
https://www.xicidaili.com/nn/6
/nn/7
https://www.xicidaili.com/nn/7
/nn/8
https://www.xicidaili.com/nn/8
/nn/9
https://www.xicidaili.com/nn/9
/nn/10
https://www.xicidaili.com/nn/10
/nn/11
https://www.xicidaili.com/nn/11
/nn/12
https://www.xicidaili.com/nn/12
/nn/13
https://www.xicidaili.com/nn/13
/nn/14
https://www.xicidaili.com/nn/14
/nn/15
https://www.xicidaili.com/nn/15
/nn/16
https://www.xicidaili.com/nn/16
/nn/17
https://www.xicidaili.com/nn/17
/nn/18
https://www.xicidaili.com/nn/18
/nn/19
https://www.xicidaili.com/nn/19
/nn/20
https://www.xicidaili.com/nn/20
/nn/21
https://www.xicidaili.com/nn/21
/nn/22
https://www.xicidaili.com/nn/22
/nn/23
https://www.xicidaili.com/nn/23
/nn/24
https://www.xicidaili.com/nn/24
/nn/25
https://www.xicidaili.com/nn/25
/nn/26
https://www.xicidaili.com/nn/26
/nn/27
https://www.xicidaili.com/nn/27
/nn/28
https://www.xicidaili.com/nn/28
/nn/29
https://www.xicidaili.com/nn/29
/nn/30
https://www.xicidaili.com/nn/30
/nn/31
https://www.xicidaili.com/nn/31
/nn/32
https://www.xicidaili.com/nn/32
/nn/33
https://www.xicidaili.com/nn/33
/nn/34
https://www.xicidaili.com/nn/34
/nn/35
https://www.xicidaili.com/nn/35
/nn/36
https://www.xicidaili.com/nn/36
/nn/37
https://www.xicidaili.com/nn/37
/nn/38
https://www.xicidaili.com/nn/38
/nn/39
https://www.xicidaili.com/nn/39
/nn/40
https://www.xicidaili.com/nn/40
/nn/41
https://www.xicidaili.com/nn/41
/nn/42
https://www.xicidaili.com/nn/42
/nn/43
https://www.xicidaili.com/nn/43
/nn/44
https://www.xicidaili.com/nn/44
/nn/45
https://www.xicidaili.com/nn/45
/nn/46
https://www.xicidaili.com/nn/46
/nn/47
https://www.xicidaili.com/nn/47
/nn/48
https://www.xicidaili.com/nn/48
/nn/49
https://www.xicidaili.com/nn/49
/nn/50
https://www.xicidaili.com/nn/50
/nn/51
https://www.xicidaili.com/nn/51
/nn/52
https://www.xicidaili.com/nn/52
/nn/53
https://www.xicidaili.com/nn/53
/nn/54
https://www.xicidaili.com/nn/54
/nn/55
https://www.xicidaili.com/nn/55
/nn/56
https://www.xicidaili.com/nn/56
/nn/57
https://www.xicidaili.com/nn/57
/nn/58
https://www.xicidaili.com/nn/58
/nn/59
https://www.xicidaili.com/nn/59
/nn/60
https://www.xicidaili.com/nn/60
/nn/61
https://www.xicidaili.com/nn/61
/nn/62
https://www.xicidaili.com/nn/62
/nn/63
https://www.xicidaili.com/nn/63
/nn/64
https://www.xicidaili.com/nn/64

```



```
cidaili.com/nn/26>
{'ip': '171.35.170.145', 'port': '9999'}
2019-09-15 22:10:32 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/27> (referer: https://www.xicidaili.com/nn/26)
2019-09-15 22:10:32 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/27>
{'ip': '120.83.106.42', 'port': '9999'}
2019-09-15 22:10:32 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/28> (referer: https://www.xicidaili.com/nn/27)
2019-09-15 22:10:32 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/28>
{'ip': '121.233.207.233', 'port': '9999'}
2019-09-15 22:10:33 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/29> (referer: https://www.xicidaili.com/nn/28)
2019-09-15 22:10:33 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/29>
{'ip': '222.189.246.212', 'port': '9999'}
2019-09-15 22:10:33 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/30> (referer: https://www.xicidaili.com/nn/29)
2019-09-15 22:10:33 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/30>
{'ip': '163.204.240.119', 'port': '9999'}
2019-09-15 22:10:33 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/31> (referer: https://www.xicidaili.com/nn/30)
2019-09-15 22:10:33 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/31>
{'ip': '222.94.28.28', 'port': '61234'}
2019-09-15 22:10:33 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/32> (referer: https://www.xicidaili.com/nn/31)
2019-09-15 22:10:33 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/32>
{'ip': '163.204.244.109', 'port': '9999'}
2019-09-15 22:10:33 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/33> (referer: https://www.xicidaili.com/nn/32)
2019-09-15 22:10:33 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/33>
{'ip': '123.101.231.172', 'port': '9999'}
2019-09-15 22:10:33 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/34> (referer: https://www.xicidaili.com/nn/33)
2019-09-15 22:10:34 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/34>
{'ip': '171.12.112.249', 'port': '9999'}
2019-09-15 22:10:34 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/35> (referer: https://www.xicidaili.com/nn/34)
2019-09-15 22:10:34 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/35>
{'ip': '112.85.129.128', 'port': '9999'}
2019-09-15 22:10:34 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/36> (referer: https://www.xicidaili.com/nn/35)
2019-09-15 22:10:34 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/36>
{'ip': '36.248.129.245', 'port': '9999'}
2019-09-15 22:10:34 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/37> (referer: https://www.xicidaili.com/nn/36)
2019-09-15 22:10:34 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/37>
{'ip': '182.35.87.222', 'port': '9999'}
2019-09-15 22:10:34 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/38> (referer: https://www.xicidaili.com/nn/37)
2019-09-15 22:10:34 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.xicidaili.com/nn/38>
{'ip': '171.12.112.83', 'port': '9999'}
2019-09-15 22:10:34 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.xicidaili.com/nn/39> (referer: https://www.xic
```

ip.csv文件：

```
ip	port
120.83.101.212	9999
ip	port
1.199.31.244	9999
171.35.221.159	9999
163.204.247.65	9999
117.85.105.247	9999
1.197.11.251	9999
171.35.169.79	9999
163.204.245.194	9999
59.32.37.229	8010
182.35.87.13	9999
120.83.108.23	9999
113.120.60.45	9999
163.204.243.223	9999
175.42.128.58	9999
112.87.69.203	9999
120.84.101.87	808
114.239.1.72	808
112.87.69.92	9999
1.197.16.182	9999
106.15.75.229	8080
175.42.129.218	9999
163.204.246.195	9999
171.35.170.145	9999
120.83.106.42	9999
121.233.207.233	9999
222.189.246.212	9999
163.204.240.119	9999
222.94.28.28	61234
163.204.244.109	9999
123.101.231.172	9999
171.12.112.249	9999
112.85.129.128	9999
36.248.129.245	9999
182.35.87.222	9999
171.12.112.83	9999
114.239.251.120	9999
163.204.240.3	9999
1.197.204.99	9999
121.234.66.13	808
163.204.243.168	9999
1.198.73.105	9999
182.34.35.181	9999
117.91.130.58	9999
144.123.71.20	20564
117.28.96.66	9999
182.35.87.246	9999
120.83.111.147	9999
121.226.214.126	9999
112.85.172.238	9999
120.83.111.39	9999
180.119.141.209	22646
114.217.218.229	8118
182.34.34.82	9999
182.35.82.173	9999
112.85.171.174	9999
60.13.42.214	9999
122.4.44.73	9999
123.169.35.62	9999
113.124.84.14	9999
182.116.225.198	9999

```

到此，就结束了！

## 5、致谢

感谢您的观看，如果觉得帮助到您的话，请点击一下广告支持一下小站，十分感谢！！！