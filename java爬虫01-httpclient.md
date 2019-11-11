---
title: java爬虫01-----httpclient
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: httpclient建立爬虫程序
categories: httpclient
tags: httpclient
keywords: 'httpclient,爬虫'
abbrlink: c2d8ad83
date: 2019-08-29 09:55:54
password:
copyright: true
---

# java爬虫01-----httpclient

## 1、准备环境

IDE: idea

jdk: jdk8

httpclient: 4.5.2

maven: maven

## 2、加入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>4.5.9</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-to-slf4j</artifactId>
        <version>2.11.2</version>
    </dependency>
</dependencies>
```

## 3、编写爬虫入门程序

```java
package top.juntech;


import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
//import org.slf4j.Logger;
//import org.slf4j.LoggerFactory;

import java.io.IOException;

public class crawlerFirst  {
//    private  static  final Logger logger = LoggerFactory.getLogger(crawlerFirst.class);
    public static void main(String[] args) throws IOException{

        String uri = "http://news.baidu.com/";

//        打开浏览器
        CloseableHttpClient client = HttpClients.createDefault();
//        输入网址
        HttpGet httpGet = new HttpGet(uri);
//        发送请求
        CloseableHttpResponse response = client.execute(httpGet);
//        接收数据
        if(response.getStatusLine().getStatusCode() == 200){
            HttpEntity entity = response.getEntity();
            String content = EntityUtils.toString(entity,"utf-8");
            System.out.println(content);
//            logger.info(response.getEntity().getContent().toString());
        }
    }

}
```

运行程序：

```html
......
<div class="hotnews" alog-group="focustop-hotnews">
<ul><li class="hdline0">
<i class="dot"></i>
<strong>
<a href="http://news.cctv.com/special/qgtz/qgtzzgjj/index.shtml" target="_blank" class="a3" mon="ct=1&a=1&c=top&pn=0">强国图志——中国经济是一片大海</a></strong>
</li>
<li class="hdline1">
<i class="dot"></i>
<strong>
<a href="http://news.cctv.com/2019/08/28/ARTIW0rZpeWcINBvgaF9jh8L190828.shtml" target="_blank"  mon="r=1">习近平接受外国新任驻华大使递交国书</a>
<i style="font-size: 12px">&nbsp;</i><a href="http://news.cctv.com/2019/08/28/ARTI7SscxrTjvy5g5arXXPV9190828.shtml" target="_blank"  mon="r=1">会见阿里波夫</a>
</strong>
</li>
<li class="hdline2">
<i class="dot"></i>
<strong>
<a href="http://www.xinhuanet.com/politics/2019-08/28/c_1124930608.htm" target="_blank" class="a3" mon="ct=1&a=1&c=top&pn=1">行动起来：共同呵护好孩子们的眼睛</a><i style="font-size: 12px">&nbsp;</i><a href="http://opinion.people.com.cn/n1/2019/0828/c1003-31321094.html" target="_blank" class="a3" mon="ct=1&a=1&c=top&pn=1">发展有温度</a>
</strong>
</li>
<li class="hdline3">
<i class="dot"></i>
<strong>
<a href="http://www.qstheory.cn/wp/2019-08/28/c_1124930153.htm" target="_blank"  mon="r=1">【中国稳健前行】厚植文化自信 增强战略定力 </a>
</strong>
</li>
    ......
```

上面是其中的源码的一部分

其中：

习近平接受外国新任驻华大使递交国书  ： href="http://news.cctv.com/special/qgtz/qgtzzgjj/index.shtml"

这些就是我们要爬取的内容

## 4、httpGetdemo

不带参数的httpclient  get请求爬虫

```java
package top.juntech;



import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

public class HttpGetDemo {
    public static void main(String[] args) throws IOException {
        String uri = "http://news.baidu.com/";
//        创建httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
//        创建httpGet对象
        HttpGet httpGet = new HttpGet(uri);
        CloseableHttpResponse response = null;
//        使用httpclient发起请求，返回resp
        try {
            response = httpclient.execute(httpGet);
            //        解析响应
            if(response.getStatusLine().getStatusCode() == 200){  //响应code为200表示可连接
                String content = EntityUtils.toString( response.getEntity(),"utf8");
                System.out.println(content.length());
            }
        }catch (Exception e){
//            throw  new Exception(e);
            e.printStackTrace();
        }finally {
//            关闭response
            try{
                response.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }

    }

}
```

控制台信息： 70663

## 5、httpGetParamdemo

带参数的httpclient  get请求爬虫

```java
package top.juntech;


import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

public class httpGetParamDemo {
    public static void main(String[] args) throws IOException {

        String uri = "http://news.baidu.com/";
//        创建httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
//        创建httpGet对象
        HttpGet httpGet = new HttpGet(uri);
        System.out.println("发起请求的信息： "+httpGet);
        CloseableHttpResponse response = null;
//        使用httpclient发起请求，返回resp
        try {
            response = httpclient.execute(httpGet);
            //        解析响应
            if(response.getStatusLine().getStatusCode() == 200){  //响应code为200表示可连接
                String content = EntityUtils.toString( response.getEntity(),"utf8");
                System.out.println(content.length());
            }
        }catch (Exception e){
//            throw  new Exception(e);
            e.printStackTrace();
        }finally {
//            关闭response
            try{
                response.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }

    }

}
```

发起请求的信息： GET http://news.baidu.com/ HTTP/1.1

ctrl+点击  打开  HttpGet源码：

```java
package org.apache.http.client.methods;

import java.net.URI;

public class HttpGet extends HttpRequestBase {
    public static final String METHOD_NAME = "GET";

    public HttpGet() {
    }

    public HttpGet(URI uri) {
        this.setURI(uri);
    }

    public HttpGet(String uri) {
        this.setURI(URI.create(uri));
    }

    public String getMethod() {
        return "GET";
    }
}
```

发现不只有HttpGet(string uri)方法还有 public HttpGet(URI uri)

查看URI源码：

```java
public final class URI
    implements Comparable<URI>, Serializable
{

    // Note: Comments containing the word "ASSERT" indicate places where a
    // throw of an InternalError should be replaced by an appropriate assertion
    // statement once asserts are enabled in the build.

    static final long serialVersionUID = -6052424284110960213L;


    // -- Properties and components of this instance --

    // Components of all URIs: [<scheme>:]<scheme-specific-part>[#<fragment>]
    private transient String scheme;            // null ==> relative URI
    private transient String fragment;

    // Hierarchical URI components: [//<authority>]<path>[?<query>]
    private transient String authority;         // Registry or server

    // Server-based authority: [<userInfo>@]<host>[:<port>]
    private transient String userInfo;
    private transient String host;              // null ==> registry-based
    private transient int port = -1;            // -1 ==> undefined

    // Remaining components of hierarchical URIs
    private transient String path;              // null ==> opaque
    private transient String query;

    // The remaining fields may be computed on demand

    private volatile transient String schemeSpecificPart;
    private volatile transient int hash;        // Zero ==> undefined

    private volatile transient String decodedUserInfo = null;
    private volatile transient String decodedAuthority = null;
    private volatile transient String decodedPath = null;
    private volatile transient String decodedQuery = null;
    private volatile transient String decodedFragment = null;
    private volatile transient String decodedSchemeSpecificPart = null;

    /**
     * The string form of this URI.
     *
     * @serial
     */
    private volatile String string;             // The only serializable field



    // -- Constructors and factories --

    private URI() { }                           // Used internally
```

httpGetParamDemo.java

```java
package top.juntech;


import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

public class httpGetParamDemo {
    public static void main(String[] args) throws Exception {

        String uri = "https://www.qidian.com/search";
//        创建httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
//        设置请求地址： https://www.qidian.com/search?kw=斗破苍穹
//        创建URIBuilder
        URIBuilder uriBuilder = new URIBuilder(uri);
//        设置参数
        uriBuilder.setParameter("kw","斗破苍穹");
        //若有多个参数
        //uriBUilder.setParameter(...).setParameter(...);
//        创建httpGet对象
        HttpGet httpGet = new HttpGet(uriBuilder.build());
        System.out.println("发起请求的信息： "+httpGet);
        CloseableHttpResponse response = null;
//        使用httpclient发起请求，返回resp
        try {
            response = httpclient.execute(httpGet);
            //        解析响应
            if(response.getStatusLine().getStatusCode() == 200){  //响应code为200表示可连接
                String content = EntityUtils.toString( response.getEntity(),"utf8");
                System.out.println(content);
            }
        }catch (Exception e){
//            throw  new Exception(e);
            e.printStackTrace();
        }finally {
//            关闭response
            try{
                response.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }

    }

}
```

控制台打印：

发起请求的信息： GET https://www.qidian.com/search?kw=%E6%96%97%E7%A0%B4%E8%8B%8D%E7%A9%B9 HTTP/1.1

```json
<link rel="stylesheet" data-ignore="true" href="//qidian.gtimg.com/c/=/qd/css/reset.ddecf.css,/qd/css/global.d41d8.css,/qd/css/font.1727c.css,/qd/css/header.17852.css,/qd/css/module.89faa.css,/qd/css/list_module.37b81.css,/qd/css/search.1d4fa.css,/qd/css/layout.5de0f.css,/qd/css/qd_popup.ed72c.css,/qd/css/footer.5ccc0.css,/qd/css/lbfUI/css/ComboBox.8997f.css,/qd/css/lbfUI/css/Button.e5a3e.css" />
<script data-ignore="true" src="//qidian.gtimg.com/lbf/1.1.0/LBF.js?max_age=31536000"></script>
<script>
    // LBF 配置
    LBF.config({"paths":{"site":"//qidian.gtimg.com/qd/js","qd":"//qidian.gtimg.com/qd","common":"//qidian.gtimg.com/common/1.0.0"},"vars":{"theme":"//qidian.gtimg.com/qd/css"},"combo":true,"debug":false});
    LBF.use(['lib.jQuery'], function ($) {
        window.$ = $;
    });
</script>
<script>
    LBF.use(['monitor.SpeedReport', 'qd/js/component/login.a4de6.js', 'qd/js/search/index.83974.js' ], function (SpeedReport, Login, Index) {
        // 页面逻辑入口
        if(Login){
            Login.init().always(function(){
                Index && typeof Index === 'function' && new Index();
            })
        }
        if(219 && 219 != ''){
            $(window).on('load.speedReport', function () {
                // speedTimer[onload]
                speedTimer.push(new Date().getTime());
                var f1 = 7718, // china reading limited's ID
                        f2 = 219, // site ID
                        f3 = 16; // page ID
                // chrome & IE9 Performance API
                SpeedReport.reportPerformance({
                    flag1: f1,
                    flag2: f2,
                    flag3IE: f3,
                    flag3Chrome: f3,
                    rate:0.1,
                    url: '//isdspeed.qidian.com/cgi-bin/r.cgi'
                });
                // common speedTimer:['dom ready', 'onload']
                var speedReport = SpeedReport.create({
                    flag1: f1,
                    flag2: f2,
                    flag3: f3,
                    start: speedZero,
                    rate:0.1,
                    url: '//isdspeed.qidian.com/cgi-bin/r.cgi'
                });
                // chrome & IE9 Performance API range 1~19, common speedTimer use 20+
                for (var i = 0; i < speedTimer.length; i++) {
                    speedReport.add(speedTimer[i], i + 20)
                }
                // http://isdspeed.qq.com/cgi-bin/r.cgi?flag1=7718&flag2=224&flag3=1&1=38&2=38&…
                speedReport.send();
            })
        }
    });
    speedTimer.push(new Date().getTime());
</script>
<script>
    var _mtac = {};
    (function() {
        var mta = document.createElement("script");
        mta.src = "//pingjs.qq.com/h5/stats.js?v2.0.2";
        mta.setAttribute("name", "MTAH5");
        mta.setAttribute("sid", "500451537");
        var s = document.getElementsByTagName("script")[0];
        s.parentNode.insertBefore(mta, s);
    })();
</script>
</html>

Process finished with exit code 0

```

## 6、HttpPostDemo

http post请求不带参数

```java
package top.juntech;

import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

public class HttpPostDemo {
    public static void main(String[] args) throws Exception{
        String uri = "https://www.qidian.com";
//        打开浏览器
        CloseableHttpClient client = HttpClients.createDefault();
//    输入网址
        HttpPost httpPost = new HttpPost(uri);
        System.out.println(httpPost);
//        发送请求 httppost
        CloseableHttpResponse resp = null;
        try{
            resp = client.execute(httpPost);
//            解析响应
            if (resp.getStatusLine().getStatusCode() == 200){
                String content = EntityUtils.toString(resp.getEntity(), "utf8");
                System.out.println(content);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                resp.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

## 7、httpPostParamDemo

http post请求带参数

```java
package top.juntech;

import org.apache.http.NameValuePair;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

import java.util.ArrayList;
import java.util.List;

public class HttpPosParamtDemo {
    public static void main(String[] args) throws Exception{
        String uri = "https://www.qidian.com/search";
        HttpPost httpPost = new HttpPost(uri);
//        打开浏览器
        CloseableHttpClient client = HttpClients.createDefault();
//    输入网址
//        声明list集合，存放表单参数
        List<NameValuePair> params = new ArrayList<NameValuePair>();
        params.add(new BasicNameValuePair("kw","斗破苍穹"));
//        创建表单实体对象
        UrlEncodedFormEntity formEntity = new UrlEncodedFormEntity(params,"utf8");
//        设置表单对象到post请求中
        httpPost.setEntity(formEntity);
        System.out.println(httpPost);
//        发送请求 httppost
        CloseableHttpResponse resp = null;
        try{
            resp = client.execute(httpPost);
//            解析响应
            if (resp.getStatusLine().getStatusCode() == 200){
                String content = EntityUtils.toString(resp.getEntity(), "utf8");
                System.out.println(content);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                resp.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```

控制台打印：

POST https://www.qidian.com/search HTTP/1.1

这里的参数不是和post并排的，是分开的另外的一个实体对象。

## 8、创建连接池

综合以上的例子，我们发现每次都需要创建和销毁HttpClient,HttpClient相当于一个浏览器，每次都要打开和关闭，参考数据库的例子，我们创建一个连接池，要用httpclient的时候在从里面获取httpclient就行了。

首先创建连接池管理器：

```java
PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
```

使用连接池管理器发起请求

```java
doGet(cm);
```

创建HttpClientPool.java

```java
package top.juntech;

import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.util.EntityUtils;

public class HttpClientPool {
    public static void main(String[] args) {
        //    创建连接池管理器
        PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
//        设置最大连接数
        cm.setMaxTotal(100);
//        设置最达连接主机数
        cm.setDefaultMaxPerRoute(10);
//    使用连接池管理器发起请求
        doGet(cm);
//        doGet(cm);
    }

    public static  void doGet(PoolingHttpClientConnectionManager cm){
//        不是每次创建新的httpcleint,二十从连接池中获取httpclient对象，不能关闭httpclient
        CloseableHttpClient client = HttpClients.custom().setConnectionManager(cm).build();

        HttpGet httpGet = new HttpGet("https://www.qidian.com/xuanhuan");

        CloseableHttpResponse resp = null;

        try {
           resp = client.execute(httpGet);
           if(resp.getStatusLine().getStatusCode() == 200){
               System.out.println(resp.getEntity());
               String content = EntityUtils.toString(resp.getEntity(), "utf8");
               System.out.println(content);
           }

        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if(resp != null) {
                try {
                    resp.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

    }

}
```

## 9、请求参数

一些配置信息，由于网络时延或者其他的一些因素导致网络不可达，或者数据传输速度太慢，需要设置一些时间参数来控制。

配置请求参数：

```java
package top.juntech;



import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;

public class HttpConfig {
    public static void main(String[] args) throws IOException {

        String uri = "http://news.baidu.com/";
//        创建httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();
//        创建httpGet对象
        HttpGet httpGet = new HttpGet(uri);
        //设置请求信息,时间单位为ms
        RequestConfig requestConfig = RequestConfig.custom().setConnectionRequestTimeout(5000)
                .setConnectTimeout(1000)
                .setSocketTimeout(10*1000)
            	.setProxy(null)
            //还有很多参数可以设置，大家可以试试
                .build();
//        传入参数
        httpGet.setConfig(requestConfig);
        CloseableHttpResponse response = null;
//        使用httpclient发起请求，返回resp
        try {
            response = httpclient.execute(httpGet);
            //        解析响应
            if(response.getStatusLine().getStatusCode() == 200){  //响应code为200表示可连接
                String content = EntityUtils.toString( response.getEntity(),"utf8");
                System.out.println(content.length());
            }
        }catch (Exception e){
//            throw  new Exception(e);
            e.printStackTrace();
        }finally {
//            关闭response
            try{
                response.close();
            }catch (Exception e){
                e.printStackTrace();
            }
        }

    }

}
```