## springboot整合redis乱码

## 1.准备环境

开发工具：idea，redisdesktop

开发环境：jdk8,redis

## 2.创建工程

在idea中创建一个springboot工程。

导入依赖：

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.juntech</groupId>
    <artifactId>springboot-redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-redis</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.1.6.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

为了方便演示，我们这里就只创建一个bean对象和一个controller，就不连接数据库哪些了。

一个book类：

```java
package top.juntech.springbootredis.bean;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

import java.io.Serializable;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/12 15:47
 * @ClassName 类名
 * @Descripe 描述
 */
@Getter
@Setter
@ToString(callSuper = true)
public class Book implements Serializable {
    private Integer id;
    private String name;
    private String author;
}

```

一个控制器 bookcontroller

```java
package top.juntech.springbootredis.controller;

import jdk.nashorn.internal.objects.annotations.Getter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import top.juntech.springbootredis.bean.Book;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/12 15:50
 * @ClassName 类名
 * @Descripe 描述
 */
@RestController
public class BookController {

    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private StringRedisTemplate stringRedisTemplate;


    @GetMapping("/test1")
    public void test1(){
        ValueOperations<String, String> ops1 = stringRedisTemplate.opsForValue();
        ops1.set("name","三国演义");
        String name = ops1.get("name");
        System.out.println("从redis中获取 到name: "+name);
        System.out.println("=============================");
        ValueOperations ops2 = redisTemplate.opsForValue();
        Book book1 = new Book();
        book1.setId(1);
        book1.setAuthor("曹雪芹");
        book1.setName("三国演义");

        ops2.set("book_"+book1.getId(),book1);
        Book book_1 = (Book)ops2.get("book_1");
        System.out.println("redistemplate 获取到book_1:"+book_1);
    }
}

```

applicaiton.properties

```
server.port=8080

spring.redis.password=
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.database=0
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.min-idle=0

```



## 3.准备工作完成，开始测试

打开浏览器，数据localhost:8080

没有报错就是成功了，查看控制台信息：

```
从redis中获取 到name: 三国演义
=============================
redistemplate 获取到book_1:Book(super=top.juntech.springbootredis.bean.Book@6f03ff70, id=1, name=三国演义, author=曹雪芹)
```

然后打开redis客户端工具，redisdesktop查看，不知道这个工具的可以自行百度下载。

打开一看，不对啊，怎么会乱码呢？

![](https://img.vim-cn.com/64/3882afaf9efc51fe96db6ba7048c5246245279.png)

因为redisTemplate默认使用的是jdkserilizationRedisSerializer序列化器，StringredisTemplate使用的是StringREdisSerializer序列化器。

## 4.解决问题

我们重写redisTempalte给它重新设置一个序列化器就ok了。

怎么做呢？

我们知道redisTemplate加上@Autowired就会自动注入了，那它是怎么自动注入的呢，我们进入RedisAutoConfiguration这个类，idea双击shift搜索就行了。

查看源码部分为：

```java
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }
       @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
```

在这里我们可以看到这个自动配置的类包含了redistempalte和StringRedistemplate，而这两个使我们需要的，现在我们来重写他们，stringredistempalte没问题就不重写它了，默认就行

新建一个包config,新建一个类RedisConfig.java

```
package top.juntech.springbootredis.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import top.juntech.springbootredis.bean.Book;

import java.net.UnknownHostException;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/12 15:58
 * @ClassName 类名
 * @Descripe 描述
 */
@Configuration
public class RedisConfig  {

   @Bean
    public RedisTemplate<String, Book> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<String, Book> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Book.class);
        template.setDefaultSerializer(jackson2JsonRedisSerializer);
        return template;
    }

}

```

在这里其他的东西我们保持默认，但是序列化器要变为json2这个，具体表现：

```
 Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Book.class);
        template.setDefaultSerializer(jackson2JsonRedisSerializer);
```

这样就行了，重新运行项目，访问接口，打开redis客户端工具，就可以看到正常的了。如下

![](https://img.vim-cn.com/70/4ad88cc29ccdb6fdfc18a25e2d2943e41c0a87.png)