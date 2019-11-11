---
title: 'google的json工具:gson'
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
summary: 'google的json工具:gson'
categories: tools
tags: tools
keywords: gson
abbrlink: 3333abd9
date: 2019-11-09 13:34:26
---

# google的json处理工具：Gson

## 1.去maven库里寻找gson，并导入pom.xml

网络直通车：<https://mvnrepository.com/artifact/com.google.code.gson/gson>

选择版本：

```xml
<!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>

```

1. 简单的 Java Object 序列化/反序列化
   序列化

假如有一个User类，拥有 name, email, age, isDeveloper 四个属性，如下：

```java
User userObject = new User(  

    "Norman", 

    "norman@futurestud.io", 

    26, 

    true

);

```


使用Gson将它序列化：

```
Gson gson = new Gson();

String userJson = gson.toJson(userObject);

```


得到的结果如下：

```
{

	"isDeveloper":true,

	"name":"Norman",

	"age":26,

	"email":"norman@futurestud.io"

}

```



反序列化

先定义一段JSON字符串

```
String userJson = "{'isDeveloper':false,'name':'xiaoqiang','age':26,'email':'578570174@qq.com'}";

1

Gson反序列化

User user = gson.fromJson(userJson, User.class);

1

```



String userJson = "{'isDeveloper':false,'name':'xiaoqiang','age':26,'email':'578570174@qq.com'}";
1
Gson反序列化

User user = gson.fromJson(userJson, User.class);
1
debug一下，查看结果
![](https://img-blog.csdn.net/20180512142304582?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW5yZW54aWFuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2. 嵌套 Java Object 的序列化/反序列化

也就是说，一个类里面还包含有其它类。比如`User`类里面还有个用户地址`UserAddress`类，JSON结构如下：

```json
{
    "age": 26,
    "email": "578570174@qq.com",
    "isDeveloper": true,
    "name": "chenrenxiang",

    "userAddress": {
        "city": "Magdeburg",
        "country": "Germany",
        "houseNumber": "42A",
        "street": "Main Street"
    }
}

```

还有很多，例如Array 和 List 的序列化/反序列化，Map 和 Set 的序列化/反序列化....

## 3.获取类的类型

使用方法：

```
new TypeToken<class>(){}.getType()
```

就这样，可以获取class的类型