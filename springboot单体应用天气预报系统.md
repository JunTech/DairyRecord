---
title: springboot单体应用天气预报系统
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: springboot 单体引用 天气预报系统的构建
categories: springboot
tags: springboot
keywords: 天气预报系统
abbrlink: e26a506c
date: 2019-08-23 12:41:18
password:
copyright: true
---

# springboot单体应用天气预报系统


数据准备：
接口：

http://wthrcdn.etouch.cn/weather_mini?citykey=101280101

http://wthrcdn.etouch.cn/weather_mini?city=杭州

数据格式：

{"data":{"yesterday":{"date":"21日星期三","high":"高温 35℃","fx":"无持续风向","low":"低温 27℃","fl":"<![CDATA[<3级]]>","type":"雷阵雨"},"city":"广州","forecast":[{"date":"22日星期四","high":"高温 35℃","fengli":"<![CDATA[<3级]]>","low":"低温 27℃","fengxiang":"无持续风向","type":"雷阵雨"},{"date":"23日星期五","high":"高温 36℃","fengli":"<![CDATA[<3级]]>","low":"低温 28℃","fengxiang":"无持续风向","type":"雷阵雨"},{"date":"24日星期六","high":"高温 36℃","fengli":"<![CDATA[<3级]]>","low":"低温 26℃","fengxiang":"无持续风向","type":"多云"},{"date":"25日星期天","high":"高温 34℃","fengli":"<![CDATA[<3级]]>","low":"低温 26℃","fengxiang":"无持续风向","type":"雷阵雨"},{"date":"26日星期一","high":"高温 34℃","fengli":"<![CDATA[<3级]]>","low":"低温 26℃","fengxiang":"无持续风向","type":"雷阵雨"}],"ganmao":"各项气象条件适宜，发生感冒机率较低。但请避免长期处于空调房间中，以防感冒。","wendu":"33"},"status":1000,"desc":"OK"}

##1、根据数据创建model实体类
创建Forecast，Weather，WeatherResponse，Yesterday四个类
用于接收数据参数

##2、创建service
创建2个接口。根据id获取data:
 ```java
 WeatherResponse getWeatherDataByCityId(String cityId);
```
根据name获取data:
```java
WeatherResponse getWeatherDataByCityName(String cityName);
```
##3、创建service实现类
实现接口
由于两个接口的方法都一样，除了uri不一样，则选择方法的重构，重新写一个方法
```java
public WeatherResponse getWeatherData(String uri){
        ResponseEntity<String> respString = restTemplate.getForEntity(uri, String.class);
        ObjectMapper objectMapper =  new ObjectMapper();
        objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        WeatherResponse weatherResponse = null;
        String strBody = null;
        if(respString.getStatusCodeValue() == 200){
            strBody =respString.getBody();
        }

        try {
            weatherResponse = objectMapper.readValue(strBody,WeatherResponse.class);
            System.out.println(weatherResponse);
        }catch (Exception e){
            e.printStackTrace();
        }
```
其中用到了jackson转java实体类的方法ObjectMapper,需要引入RestTemplate
##4、创建配置类
创建java类：config.restconfig
```java
package top.juntech.singleweatherforecastsys.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestConfig {
//用到了http的方法，则需要引入RestTemplateBuilder
   @Autowired
    private RestTemplateBuilder builder;

    @Bean
    public RestTemplate restTemplate(){
        return builder.build();
    }

}
```
##5、运行程序
至此，程序数据接口就弄好了，打开Rester（或postman）测试接口，看返回数据是否发生异常
localhost:8080/weather/cityId/101280101
```java
{
    "status": "1000",
    "desc": "OK",
    "data": {
        "yesterday": {
            "high": "高温 35℃",
            "fx": "无持续风向",
            "low": "低温 27℃",
            "fl": "<![CDATA[<3级]]>",
            "type": "雷阵雨",
            "date": "22日星期四"
        },
        "city": "广州",
        "forecast": [
            {
                "date": "23日星期五",
                "high": "高温 36℃",
                "low": "低温 28℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "多云",
                "fengxiang": "无持续风向"
            },
            {
                "date": "24日星期六",
                "high": "高温 36℃",
                "low": "低温 26℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "多云",
                "fengxiang": "无持续风向"
            },
            {
                "date": "25日星期天",
                "high": "高温 34℃",
                "low": "低温 26℃",
                "fengli": "<![CDATA[3-4级]]>",
                "type": "中雨",
                "fengxiang": "南风"
            },
            {
                "date": "26日星期一",
                "high": "高温 34℃",
                "low": "低温 27℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "雷阵雨",
                "fengxiang": "无持续风向"
            },
            {
                "date": "27日星期二",
                "high": "高温 35℃",
                "low": "低温 27℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "多云",
                "fengxiang": "无持续风向"
            }
        ],
        "ganmao": "各项气象条件适宜，发生感冒机率较低。但请避免长期处于空调房间中，以防感冒。",
        "wendu": "32"
    }
}
```
则后台数据访问成功，接下来进行redis，及定时任务的接入。。。
