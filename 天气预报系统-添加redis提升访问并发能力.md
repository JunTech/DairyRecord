---
title: 天气预报系统-添加redis提升访问并发能力
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: '给天气预报系统集成redis,提升该系统的访问并发能力'
categories: redis
tags: redis
keywords: redis
abbrlink: 883e5b53
date: 2019-08-23 16:50:20
password:
copyright: true
---

添加redis来提升天气预报系统的并发访问能力



1、为什么要使用redis:

1. 	及时响应
2.          有效减少服务调用

开发环境：

	1、jdk8

	2、maven

	3、redis4.*

	4.	apache httpclient

	5、	springboot web starter

	6、spring boot data starter redis starter

接下来集成redis

上一步我们已经创建了一个单体天气预报系统应用，我们接着用！

2、集成redis

我们要使用redis,则需要导入RedisTemplate,StringRedisTemplate

找到数据访问层，进行redis的缓存操作。

首先判断是否存在缓存，如果存在，直接从缓存取，不存在，就调用服务，并存入缓存

由于service层进行数据访问的方法只有一个，就是

    public WeatherResponse getWeatherData(String uri)

则我们只需要对该方法进行数据访问的地方进行判断是否有缓存。为了方便显示，我们加入了日志系统slf4j

代码如下：

        private static final Logger logger = LoggerFactory.getLogger(WeatherDataServiceImpl.class);
        private static final String WEATHER_URI = "http://wthrcdn.etouch.cn/weather_mini?";
        private static final long TIME_OUT = 1800L;    
    @Autowired
        private StringRedisTemplate stringRedisTemplate;

    public WeatherResponse getWeatherData(String uri){
            String key = uri;
            String strBody = null;

            ObjectMapper objectMapper =  new ObjectMapper();
            objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
            WeatherResponse weatherResponse = null;
            ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
    //        先查缓存，有缓存直接在缓存中取数据
            if(stringRedisTemplate.hasKey(key)){
                logger.info("redis has data!");
               strBody = ops.get(key);
            }else {
    //        缓存没有，调用服务，获取数据，并加入缓存
    //            ResponseEntity<String> respString = restTemplate.getForEntity(uri, String.class);
    //            ObjectMapper objectMapper =  new ObjectMapper();
    //            objectMapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
    //            WeatherResponse weatherResponse = null;
                logger.info("redis don't have data!");
                ResponseEntity<String> respString = restTemplate.getForEntity(uri, String.class);
                if (respString.getStatusCodeValue() == 200) {
                    strBody = respString.getBody();
                }
    //                将数据写入缓存
                ops.set(key, strBody, TIME_OUT, TimeUnit.SECONDS);

            }
                try {
                    weatherResponse = objectMapper.readValue(strBody,WeatherResponse.class);
                    System.out.println(weatherResponse);
                }catch (Exception e){
    //                e.printStackTrace();
                    logger.error("error",e);
                }

            return weatherResponse;
        }

application.properties文件配置：
```java
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
## 连接池最大连接数（使用负值表示没有限制）
#spring.redis.pool.max-active=200
## 连接池最大阻塞等待时间（使用负值表示没有限制）
#spring.redis.pool.max-wait=-1
## 连接池中的最大空闲连接
#spring.redis.pool.max-idle=10
## 连接池中的最小空闲连接
#spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
#spring.redis.timeout=1000

```

其中参数讲解：

TIME_OUT: redis缓存过期时间

Logger: slf4j日志系统

我们只需要保存序列化为string就行了，所以我们这里采用StringRedisTemplate

这里说一下redistemplate 与stringredistempalte的区别和联系：

RedisTemplate和StringRedisTemplate的区别：

\	1. 两者的关系是StringRedisTemplate继承RedisTemplate。

\	2. 两者的数据是不共通的；也就是说StringRedisTemplate只能管理StringRedisTemplate里面的数据，			RedisTemplate只能管理RedisTemplate中的数据。

\	3. SDR默认采用的序列化策略有两种，一种是String的序列化策略，一种是JDK的序列化策略。

	StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。

	RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

由于我原来使用RedisTemplate遭遇过各种序列化及乱码问题，虽然他是泛型类，功能更全，但是我选择StringRedisTemplate,自己选择食用！！！

我们这里使用RedisWindowsDestop作为操作redis数据库的软件

传送门：http://www.downza.cn/soft/210734.html

3、启动application

注：如果发现没写入redis数据库的话，请在application上面加上注解@Enablecashing

打开<http://localhost:8080/weather/cityName/杭州

刚开始提示：redis don`t have data !

    2019-08-23 16:42:49.478  INFO 2372 --- [nio-8080-exec-9] t.j.s.s.impl.WeatherDataServiceImpl      : redis don't have data!

    WeatherResponse(status=1000, desc=OK, data=Weather(yesterday=Yesterday(high=高温 35℃, fx=东北风, low=低温 25℃, fl=<![CDATA[3-4级]]>, type=晴, date=22日星期四), city=杭州, forecast=[Forecast(date=23日星期五, high=高温 34℃, low=低温 26℃, fengli=<![CDATA[3-4级]]>, type=小雨, fengxiang=东北风), Forecast(date=24日星期六, high=高温 34℃, low=低温 27℃, fengli=<![CDATA[4-5级]]>, type=小雨, fengxiang=东北风), Forecast(date=25日星期天, high=高温 35℃, low=低温 26℃, fengli=<![CDATA[4-5级]]>, type=小雨, fengxiang=东风), Forecast(date=26日星期一, high=高温 36℃, low=低温 27℃, fengli=<![CDATA[<3级]]>, type=多云, fengxiang=无持续风向), Forecast(date=27日星期二, high=高温 37℃, low=低温 27℃, fengli=<![CDATA[<3级]]>, type=多云, fengxiang=无持续风向)], ganmao=各项气象条件适宜，发生感冒机率较低。但请避免长期处于空调房间中，以防感冒。, wendu=29))


1、将控制台清空，重新刷新页面

显示： redis has data!

    WeatherResponse(status=1000, desc=OK, data=Weather(yesterday=Yesterday(high=高温 35℃, fx=东北风, low=低温 25℃, fl=<![CDATA[3-4级]]>, type=晴, date=22日星期四), city=杭州, forecast=[Forecast(date=23日星期五, high=高温 34℃, low=低温 26℃, fengli=<![CDATA[3-4级]]>, type=小雨, fengxiang=东北风), Forecast(date=24日星期六, high=高温 34℃, low=低温 27℃, fengli=<![CDATA[4-5级]]>, type=小雨, fengxiang=东北风), Forecast(date=25日星期天, high=高温 35℃, low=低温 26℃, fengli=<![CDATA[4-5级]]>, type=小雨, fengxiang=东风), Forecast(date=26日星期一, high=高温 36℃, low=低温 27℃, fengli=<![CDATA[<3级]]>, type=多云, fengxiang=无持续风向), Forecast(date=27日星期二, high=高温 37℃, low=低温 27℃, fengli=<![CDATA[<3级]]>, type=多云, fengxiang=无持续风向)], ganmao=各项气象条件适宜，发生感冒机率较低。但请避免长期处于空调房间中，以防感冒。, wendu=29))



2、打开rediswindowsdesktop

连接redis数据库，reload一下

发现database0 有杭州的uri信息及天气预报的信息

则大功告成！

4、congratulations！！！！！
