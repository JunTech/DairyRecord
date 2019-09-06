# java爬虫02-----Jsoup

## 1、环境准备

IDE: IDEA

Maven: Maven

Jsoup1.10.2： jsoup

junit：单元测试

commons-io: 操作文件io的

commons-lang3: 使用到工具类StringUtils

pom.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.juntech</groupId>
    <artifactId>crawler</artifactId>
    <version>1.0-SNAPSHOT</version>

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
        <dependency>
            <groupId>org.jsoup</groupId>
            <artifactId>jsoup</artifactId>
            <version>1.10.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.8.1</version>
        </dependency>
    </dependencies>

</project>
```

## 2、jsoup主要功能

1. 从一个url，文件，字符串中解析HTML
2. 使用DOM或css选择器来查找、取出数据
3. 可操作html元素、属性、文本



## 3、jsoup解析URL

获取百度(www.baidu.com)的title名称：

```java
    @Test
    public  void testUri() throws Exception{
//        解析url地址，第一个参数是访问的uri,第二个参数是访问时候的超时时间
        Document document = Jsoup.parse(new URL("http://www.baidu.com"), 1000);
//        使用标签表达式，获取title
        String title = document.getElementsByTag("title").first().text();
//        打印title
        System.out.println(title);
    }
```

控制台打印：  百度一下，你就知道



## 4、jsoup解析文件

1. 准备html文件: 去百度首页，然后ctrl + s，保存网页，记录保存的位置
2. 解析html文件： 使用jsoup.parse解析文件，并设置charset
3. 按照元素或者标签什么的获取html信息

```java
@Test
public void testFile() throws  Exception{
    Document document = Jsoup.parse(new File("C:\\Users\\Ryan\\Desktop\\test.html"), "utf8");
    String title = document.getElementsByTag("title").first().text();
    System.out.println(title);
}
```

控制台打印： 百度一下，你就知道

## 5、jsoup操作DOM

元素获取：

1. 根据id查询： getElementById
2. 根据标签：   getElementByTag
3. 根据class:     getElementByClass
4. 根据属性：   getelementByAttribute

目标：<div id="head"><div id="u"><a class="toindex"  百度首页</a>

百度首页在： id =head--->id =u---->class=toindex下

```java
@Test
public void testDom() throws Exception{
    //解析文件，获取document对象
    Document document = Jsoup.parse(new File("C:\\Users\\Ryan\\Desktop\\test.html"), "utf8");
    String index = document.getElementById("head").getElementById("u").getElementsByClass("toindex").text();
    System.out.println(index);
}
```

控制台信息：  百度首页

## 6、jsoup操作数据

```java
@Test
    public void testData() throws Exception{
//        获取数据
        Document docuemnt = Jsoup.parse(new File("C:\\Users\\Ryan\\Desktop\\test.html"), "utf8");
//        元素值
        Element element = docuemnt.getElementById("s_tab");
        //使用str来保存元素值
        String str = "";
//        获取id   == s_tab
        str = element.id();
//        获取classname = s_tab
        str = element.className();
        System.out.println(str);
    }
```

## 7、jsoup使用选择器获取元素

```java
@Test
    public void testSelect() throws Exception{
//        id    el#id   :元素+id
        Document docuemnt = Jsoup.parse(new File("C:\\Users\\Ryan\\Desktop\\test.html"), "utf8");
        //el.class: 元素+ class  :  li.class_a
        //el[attr]: 元素+属性名   ： span[abc]
        //parent  > child 查询父元素下的某个直接子元素
        //parent   > * 查询父元素下的全部直接子元素
        Element element = docuemnt.select("div#head").first();

        System.out.println("获取到的内容是： "+element.text());
    }
```

控制台： 获取到的内容是： 输入法 手写 拼音 关闭 百度首页设置登录 新闻hao123地图视频贴吧学术登录设置更多产品





