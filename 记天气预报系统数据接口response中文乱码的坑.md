# 记天气预报系统数据接口response中文乱码的坑

​	开发微服务天气预报系统的过程中，遇到了很多的坑，但最让人头疼的是存入redis缓存时，中文乱码及控制台乱码的问题，这着实让人头疼，所幸，在各种社区的帮助下，成功解决了这个问题，接下来记录一下这个心酸历程！

​	本次使用的天气数据接口Api：localhost:10087/weather/cityName/重庆

​	返回数据格式：	

```json
{
    "status": "1000",
    "desc": "OK",
    "data": {
        "yesterday": {
            "high": "高温 38℃",
            "fx": "无持续风向",
            "low": "低温 30℃",
            "fl": "<![CDATA[<3级]]>",
            "type": "晴",
            "date": "25日星期日"
        },
        "city": "重庆",
        "forecast": [
            {
                "date": "26日星期一",
                "high": "高温 39℃",
                "low": "低温 30℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "晴",
                "fengxiang": "无持续风向"
            },
            {
                "date": "27日星期二",
                "high": "高温 35℃",
                "low": "低温 28℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "多云",
                "fengxiang": "无持续风向"
            },
            {
                "date": "28日星期三",
                "high": "高温 30℃",
                "low": "低温 26℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "中雨",
                "fengxiang": "无持续风向"
            },
            {
                "date": "29日星期四",
                "high": "高温 31℃",
                "low": "低温 24℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "阵雨",
                "fengxiang": "无持续风向"
            },
            {
                "date": "30日星期五",
                "high": "高温 32℃",
                "low": "低温 25℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "阴",
                "fengxiang": "无持续风向"
            }
        ],
        "ganmao": "各项气象条件适宜，发生感冒机率较低。但请避免长期处于空调房间中，以防感冒。",
        "wendu": "34"
    }
}
```

通过浏览器返回数据都正常，但是已存入缓存就不行了，各种乱码。

一开始，我以为是编辑器的编码错误，然后就修改idea的编码：setting-->file encoding---->全部改为utf-8

结果：不行

后来，我想了一下，毕竟调用了RestTemplate作为http请求第三方api，所以在resttempalte上面找问题，打断点调试，发现他的编码和解析的编码不一致。

```
StringHttpMessageConverter 默认是 ISO_8859_1，所以我们设置为了 UTF_8。
然后新建配置类RestConfig:
```

```java
package top.juntech.msweatherdata.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.web.client.RestTemplate;

import java.nio.charset.StandardCharsets;


@Configuration
public class RestConfig {
    @Bean
    public RestTemplate restTemplate(){
        RestTemplate restTemplate = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
        restTemplate.getMessageConverters().set(1,new StringHttpMessageConverter(StandardCharsets.UTF_8));
        return new RestTemplate();
    }

}
```

怀着满心欢喜，心想终于可以成功了吧.....

结果：还是不行

最后找到数据返回的消息头：

发现数据是经过 GZIP 压缩过的。默认情况下， RestTemplate 使用的是 JDK 的 HTTP 调用器，并不支持 GZIP 解压，难怪解析不了。



### 解决方案

#### 	1、新建StringUtil

```java
package top.juntech.msweatherdata.utils;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.zip.GZIPInputStream;

public class StringUtils {
    /**
     * 处理 Gizp 压缩的数据.
     *
     * @param str
     * @return
     * @throws IOException
     */
    public static String conventFromGzip(String str) throws IOException {
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ByteArrayInputStream in;
        GZIPInputStream gunzip = null;

        in = new ByteArrayInputStream(str.getBytes("ISO-8859-1"));
        gunzip = new GZIPInputStream(in);
        byte[] buffer = new byte[256];
        int n;
        while ((n = gunzip.read(buffer)) >= 0) {
            out.write(buffer, 0, n);
        }

        return out.toString();
    }

}
```

在核心逻辑里面：

```java
    public WeatherResponse dogetWeatherResponse(String uri){
        WeatherResponse weatherResponse = null;
        String key = uri;
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
        ObjectMapper objectMapper = new ObjectMapper();
        String strBody = null;
//        先查询是否有缓存
        if(stringRedisTemplate.hasKey(key)){
            logger.info("redis has data!");
            strBody = ops.get(key);
        }else{
            logger.info("redis don't have data!");
//            throw new RuntimeException("缓存中没有该数据!");
            ResponseEntity<String> respBody = restTemplate.getForEntity(key, String.class);
//            ResponseEntity<WeatherResponse> respBody = restTemplate.getForEntity(key, WeatherResponse.class);
            if(respBody.getStatusCodeValue() == 200){
                try {
                    strBody = StringUtils.conventFromGzip(respBody.getBody());
                }catch (Exception e){
                    e.printStackTrace();
                }

            }
//            将数据存入缓存
            ops.set(key,strBody,TIME_OUT, TimeUnit.SECONDS);
        }

        try{
            weatherResponse = objectMapper.readValue(strBody, WeatherResponse.class);
            System.out.println(weatherResponse);
        }catch (Exception e){
//            throw new RuntimeException(e);
            logger.error("error!",e);
        }
        return weatherResponse;
    }
```

结果：

​	200,ok

成功返回：

{
    "status": "1000",
    "desc": "OK",
    "data": {
        "yesterday": {
            "high": "高温 38℃",
            "fx": "无持续风向",
            "low": "低温 30℃",
            "fl": "<![CDATA[<3级]]>",
            "type": "晴",
            "date": "25日星期日"
        },
        "city": "重庆",
        "forecast": [
            {
                "date": "26日星期一",
                "high": "高温 39℃",
                "low": "低温 30℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "晴",
                "fengxiang": "无持续风向"
            },
            {
                "date": "27日星期二",
                "high": "高温 35℃",
                "low": "低温 28℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "多云",
                "fengxiang": "无持续风向"
            },
            {
                "date": "28日星期三",
                "high": "高温 30℃",
                "low": "低温 26℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "中雨",
                "fengxiang": "无持续风向"
            },
            {
                "date": "29日星期四",
                "high": "高温 31℃",
                "low": "低温 24℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "阵雨",
                "fengxiang": "无持续风向"
            },
            {
                "date": "30日星期五",
                "high": "高温 32℃",
                "low": "低温 25℃",
                "fengli": "<![CDATA[<3级]]>",
                "type": "阴",
                "fengxiang": "无持续风向"
            }
        ],
        "ganmao": "各项气象条件适宜，发生感冒机率较低。但请避免长期处于空调房间中，以防感冒。",
        "wendu": "34"
    }
}

#### 2、httpclient

 2.1 引入httpclient

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.9</version>
</dependency>
```

 2.2 编写RestTemplate

```java
@Bean
public RestTemplate restTemplate(){
    RestTemplate restTemplate = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
    restTemplate.getMessageConverters().set(1,new StringHttpMessageConverter(StandardCharsets.UTF_8));
    return new RestTemplate();
}
```

2.3 编写核心业务逻辑

```java
private WeatherResponse doGetWeatherData(String uri) {

    ResponseEntity<String> response = restTemplate.getForEntity(uri, String.class);

    String strBody = null;

    if (response.getStatusCodeValue() == 200) {
        strBody = response.getBody();
    }

    ObjectMapper mapper = new ObjectMapper();
    WeatherResponse weather = null;

    try {
        weather = mapper.readValue(strBody, WeatherResponse.class);
    } catch (IOException e) {
        e.printStackTrace();
    }

    return weather;
}
```

