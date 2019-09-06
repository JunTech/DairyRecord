# springcloud外卖订餐系统 --- day1

## 1、确定项目需求

客户端：针对普通用户--->用户登录，注册，登出，我的订单，订购菜品

后台管理系统： 针对管理员----> 管理员登陆，登出，菜单的crud操作，订单的处理，用户的crud操作

根据业务拆分微服务：(服务提供者)

account：提供账号服务，用户和管理员的登陆登出

menu: 提供菜单服务，菜单的crud操作

order: 提供订单服务，订单的crud

user： 提供用户服务，用户的crud

分离出一个服务消费者，调用以上4个服务提供者，服务消费者包含了客户端的前端页面和后台接口，后台管理系统的前端页面及后台接口，用户/管理员要访问的资源	都保存在服务消费者中，通过feign实现负载均衡

4个服务提供者及一个服务消费者都要在注册中心进行注册。配置中心也可以在其中进行统一配置管理。

## 2、创建父工程

pom.xml文件

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>starter-parent</artifactId>
    <!--<version>2.0.8.RELEASE</version>-->
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.1.6.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.8</version>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 3、注册中心

eureka-server   pom.xml

```xml
<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
</properties>

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



主程序类：application.java

```java
package top.juntech.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
//使用@EnableDiscoveryclient开启注册中心
@EnableDiscoveryClient
public class EurekaserverApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaserverApplication.class, args);
    }

}
```

配置文件：application.yml

```yml
server:
  port: 8761
eureka:
  client:
    fetch-registry: false
    register-with-eureka: false
    service-url:
      defaultZone: http://locahost:8761/eureka/
```



到此，就搭建好了一个注册中心了！

打开localhost:8761就可以看到eureka 注册中心！



## 4、配置中心

1.pom.xml

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

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

application.yml

```yaml
server:
  port: 8762
spring:
  application:
    name: eurekaserverconfig
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/shared
```

在shared文件夹创建服务提供者的配置文件：order-dev.yml

port: 

​	server: 8100

启动类

```java
package top.juntech.eurekaserverconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class EurekaserverconfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaserverconfigApplication.class, args);
    }

}

```

## 5、服务提供者order

pom.xml

```xml
<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
</properties>

<dependencies>
    <!-- 把它注册成为一个服务提供者 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
   <!--通过configserver去配置中心读取配置文件 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
    
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
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



bootstrap.yml

```yaml
spring:
  application:
    name: order
  profiles:
    active: dev
#可找到shared/order-dev.yml文件
  cloud:
    config:
      uri: http://localhost:8762
      fail-fast: true
```

ordercontroller.java

```java
package top.juntech.order.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/order")
public class OrderController {

    @Value("${server.port}")
    private String port;

    @GetMapping("/index")
    public String index(){
        return "order服务的端口："+this.port;
    }
}
```

启动类

```java
package top.juntech.order;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

}
```

第一天项目结构如下：

![56630828588](C:\Users\Ryan\AppData\Local\Temp\1566308285882.png)

依次开启Eurekaserver、eurekaserverconfig、order,会发现<http://localhost:8761/>

![56630736460](C:\Users\Ryan\AppData\Local\Temp\1566307364601.png)

![56630740034](C:\Users\Ryan\AppData\Local\Temp\1566307400341.png)