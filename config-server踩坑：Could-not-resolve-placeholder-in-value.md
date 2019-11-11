---
title: config server踩坑：Could not resolve placeholder*  in value *
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: config server踩坑记录：Could not resolve placeholder*  in value *
categories: springcloud
tags: springcloud
keywords: springcloud
abbrlink: 19efda85
date: 2019-08-27 16:09:26
password:
copyright: true
---

# config client踩坑记录--- Could not resolve placeholder*  in value *

## 1.准备环境

​	eureka server: 注册中心

​	config server: 配置中心

​	config client: 配置客户端

## 2、开始踩坑

eureka server搭建不用说，很简单，引入netflix-eureka-server依赖就行了，然后配置注册中心，主程序加入注解就ok！

pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

applicaiton.properties:

```properties
server.port= 8761
spring.application.name= eureka-server
eureka.client.service-url.default-zone = http://localhost:8761/eureka/
eureka.client.registry-fetch-interval-seconds=5000
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

applcation主程序：

```java
package top.juntech.springcloudeurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class SpringcloudEurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudEurekaServerApplication.class, args);
    }

}
```

ok,eurekaserver就搭建好了，接下来我们搭建配置中心config-server

新建module ---->config-server--->引入config  server依赖-------->配置文件-------》主程序加入注解

1、pom.xml

```xml
<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

2、application.yml(注释掉的部分是把配置中心文件放在git仓库的，为了方便，我们这里使用的是本地)

```yml
#server.port= 8763
#spring.application.name= config-server
#
##配置中心
#spring.cloud.config.server.git.uri= https://github.com/JunTech/springcloud-config-repo
#spring.cloud.config.server.git.password=
#spring.cloud.config.server.git.username=
#spring.cloud.config.server.git.search-paths=config-repo
#spring.cloud.config.label=master
server:
  port: 8763
spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/configRepo
```

3、在application.(*)文件当前目录下建立configRepo文件夹

search-locations: classpath:/configRepo 的意思是在configRepo文件夹下查找config client的配置文件

我们这里在文件夹configRepo中创建了一个文件：

config-client-dev.yml：

```yml
author: juntech
server:
  port: 10007
```

注意，我把端口信息放在配置中心的！

4、主程序加入注解，声明为配置中心

```java
package top.juntech.springcloudconfigserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
//加入注解声明为配置中心
@EnableConfigServer
public class SpringcloudConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudConfigServerApplication.class, args);
    }

}
```

ok,配置中心我们也弄完了！

最后，到config client环节了！！！！

激动否。。。

**config client搭建**

流程和上面的一毛一样。

新建module ---->config-client--->引入config  client、eureka client依赖-------->配置文件-------》主程序加入注解

1、pom.xml引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.juntech</groupId>
    <artifactId>springcloud-config-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springcloud-config-client</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
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
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

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

2、bootstrap.yml(注意，这里必须是bootstrap.yml,不然会报错)

```yml
spring:
  application:
    name: config-client
  profiles:
    active: dev
  cloud:
    config:
      uri: http://localhost:8763
eureka:
  client:
    service-url:
      default-zone: http://localhost:8761/eureka/
```

3、主程序

```java
package top.juntech.springcloudconfigclient;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
@EnableDiscoveryClient
public class SpringcloudConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringcloudConfigClientApplication.class, args);
    }

    @Value("${author}")
    private String author;

    @Value("${server.port}")
    private String port;

    @GetMapping("/hi")
    public String hi(){
        return "hi ,author : "+this.author+",i am from port : "+this.port;
    }
}
```

## 3、总结

上面的3个module按照eurekaserver--->config server---->config client依次运行，在打开localhost:8761

显示如下：

**Instances currently registered with Eureka**

| Application       | AMIs        | Availability Zones | Status                                                       |
| ----------------- | ----------- | ------------------ | ------------------------------------------------------------ |
| **CONFIG-CLIENT** | **n/a** (1) | (1)                | **UP** (1) - [Ryan:config-client:10007](http://ryan:10007/actuator/info) |

说明将config client注册到注册中心了，然后打开：<http://localhost:10007/hi>

```json
hi ,author : juntech,i am from port : 10007
```

成功了。

总结如下：

​	发生Could not resolve placeholder*  in value *错误的可能原因有以下几种：

- ​	配置中心profiles指定错误：

  ```yml
  spring:
    application:
      name: config-server
    profiles:
      active: native
  ```

  在这里，我每次都是写到spring.profiles就回车，显示就成了

  ```yml
  spring:
    application:
      name: config-server
    profiles: native  （x）
    这是错误的
  ```

- config client必须每个配置文件都必须以 bootstrap.yml命名

  之前我使用的是applicaiton.yml作为config client的配置文件，然后无一例外的都获取不到配置中心配置的server.port，每次运行都报错显示

  ```json
  启动 config-client报错 Could not resolve placeholder 'server.port' in value "${server.port}"
  ```

若还不能解决你的问题，可重构项目，参考我上面的代码，进行配置中心的config client的搭建！

## 4、参考资料

- [spring-cloud-config](https://www.springcloud.cc/spring-cloud-config.html)
- [springcloud](https://www.springcloud.cc/)
