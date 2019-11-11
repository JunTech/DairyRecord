---
title: springcloud外卖订餐系统-day2
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: menu服务及消费者feign的整合
categories: springcloud
tags: springcloud
keywords: springcloud
abbrlink: 18a81240
date: 2019-08-21 16:37:43
password:
copyright: true
---

# springcloud外卖订餐系统-day2

本次要整合mybatis,需要用到数据库，数据库的安装就不再此处讲述了，默认已安装，

直接献上sql文件：

链接: https://pan.baidu.com/s/1pyQ_zIa-_PoVPKqaIy_RDA 提取码: 53cg 复制这段内容后打开百度网盘手机App，操作更方便哦

## 1、创建模块（服务提供者）menu

创建服务提供者menu：

1、pom.xml

```xml
properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--到配置中心进行配置文件的读取-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
    <!--注册到注册中心成为服务提供者-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--mybatis-->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.0.1</version>
    </dependency>
    <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper-spring-boot-starter</artifactId>
        <version>1.1.5</version>
    </dependency>
    <!--<dependency>-->
        <!--<groupId>com.baomidou</groupId>-->
        <!--<artifactId>mybatis-plus-boot-starter</artifactId>-->
        <!--<version>3.1.2</version>-->
    <!--</dependency>-->

    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
        <!--<version>5.6.0</version>-->
    </dependency>

    <!---->

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
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

2、创建bootstrap.yml

```yaml
spring:
  application:
    name: menu
  profiles:
    active: dev
  cloud:
    config:
      uri: http://localhost:8762
      fail-fast: true
```

这里application.name指的是服务名

profiles.active指的是应用哪个环境，一般有生产环境，开发环境dev,测试环境等等

配置好了就可以访问我们第一天创建的在classpath: shared文件夹下的menu-dev.yml了

在配置中心的shared文件夹下创建menu-dev.yml

```yml
server:
  port: 8020

spring:
  datasource:
    username: root
    password: 123456
#    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:13306/orderingsystem?serverTimezone=UTC
```

创建实体类Menu.java:

```java
package top.juntech.menu.entity;


import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Menu {
    private long id;
    private String name;
    private double price;
    private String flavor;

}
```

创建数据访问层repository：

```java
package top.juntech.menu.repository;

import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;
import org.springframework.web.bind.annotation.PathVariable;
import top.juntech.menu.entity.Menu;

@Repository
public interface MenuRepository{

    @Select("select * from t_menu where id = #{id}")
    public Menu findById(@PathVariable("id")long id);

}
```

创建控制器MenuController.java:

```java
package top.juntech.menu.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;
import top.juntech.menu.entity.Menu;
import top.juntech.menu.repository.MenuRepository;


@RestController
@RequestMapping("/menu")
public class MenuController {

    @Autowired
    private MenuRepository menuRepository;

    @Value("${server.port}")
    private String port;

    @GetMapping("/index")
    public String index(){
        return "menu服务提供者的端口为："+this.port;
    }

    @GetMapping("/findbyid/{id}")
    public Menu findById(@PathVariable("id")long id){
        return menuRepository.findById(id);
    }
}
```

创建启动类：

```java
package top.juntech.menu;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@MapperScan("top.juntech.menu.repository")
@EnableDiscoveryClient
public class MenuApplication {

    public static void main(String[] args) {
        SpringApplication.run(MenuApplication.class, args);
    }

}
```



按照Eurekaserver->eurekaserverconfig->服务提供者的顺序依次执行，打开localhost:8761，就会发现有2个服务提供者创建了：

register with eureka

| Application | AMIs        | Availability Zones | Status                                                       |
| ----------- | ----------- | ------------------ | ------------------------------------------------------------ |
| **MENU**    | **n/a** (1) | (1)                | **UP** (1) - [DESKTOP-ODMEB8A:menu:8020](http://desktop-odmeb8a:8020/actuator/info) |
| **ORDER**   | **n/a** (1) | (1)                | **UP** (1) - [DESKTOP-ODMEB8A:order:8010](http://desktop-odmeb8a:8010/actuator/info) |

打开<http://localhost:8020/menu/findbyid/1>：

显示：{"id": 1,"name": "香酥鸡","price": 39,"flavor": "五香"}，则成功了

打开<http://localhost:8020/menu/findall/2/3>

[{"id": 3,"name": "栗子三杯鸡","price": 56,"flavor": "五香"},{"id": 4,"name": "毛血旺","price": 50,"flavor": "麻辣"},{"id": 5,"name": "菠菜拌粉丝","price": 22,"flavor": "五香"}]

打开<http://localhost:8020/menu/findall/0/10>

[{"id": 1,"name": "香酥鸡","price": 39,"flavor": "五香"},{"id": 2,"name": "烧椒扣肉","price": 46,"flavor": "微辣"},{"id": 3,"name": "栗子三杯鸡","price": 56,"flavor": "五香"},{"id": 4,"name": "毛血旺","price": 50,"flavor": "麻辣"},{"id": 5,"name": "菠菜拌粉丝","price": 22,"flavor": "五香"},{"id": 6,"name": "凉拌豆腐皮","price": 19,"flavor": "微辣"},{"id": 7,"name": "酱牛肉","price": 36,"flavor": "麻辣"},{"id": 8,"name": "鱼头豆腐汤","price": 32,"flavor": "五香"},{"id": 9,"name": "瘦肉鸡蛋白菜汤","price": 30,"flavor": "五香"},{"id": 10,"name": "西葫芦虾仁蒸饺","price": 26,"flavor": "五香"}

若发现在运行过程中有数据库方面的错误，请参考本站的：mabtis整合踩坑记录 这篇文章！



## 2、服务消费者整合menu



创建module  client

pom.xml

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
    <artifactId>client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>client</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR2</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
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

bootstrap.yml

```yaml
spring:
  application:
    name: client
  profiles:
    active: dev
  cloud:
    config:
      uri: http://localhost:8762
      fail-fast: true
```

client-dev.yml

```yaml
server:
  port: 8030

spring:
  application:
    name: client
  thymeleaf:
    prefix: classpath:/static/
    suffix: .html
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    instance:
      prefer-ip-address: true
```

把entity里的menu复制到client里面

创建feign-------》MenuFeign.java

```java
package top.juntech.client.feign;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import top.juntech.client.entity.Menu;

import java.util.List;

@FeignClient(value = "menu")
public interface MenuFeign {

    //获取menu的接口
    @GetMapping("/menu/findall/{index}/{limit}")
    public List<Menu> findall(@PathVariable("index") int index,@PathVariable("limit") int limit);

}
```

​	

创建ClientController.java

```java
package top.juntech.client.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import top.juntech.client.entity.Menu;
import top.juntech.client.feign.MenuFeign;

import javax.annotation.Resource;
import java.util.List;

@RestController
@RequestMapping("/client")
public class ClientController {

    @Resource
    private MenuFeign menuFeign;
//测试是否可以调用menu的接口
    @GetMapping("/client/{index}/{limit}")
    public List<Menu> findall(@PathVariable("index") int index, @PathVariable("limit") int limit){
        return menuFeign.findall(index,limit);
    }
}
```



打开<http://localhost:8030/client/client/0/10>

[{"id": 1,"name": "香酥鸡","price": 39,"flavor": "五香"},{"id": 2,"name": "烧椒扣肉","price": 46,"flavor": "微辣"},{"id": 3,"name": "栗子三杯鸡","price": 56,"flavor": "五香"},{"id": 4,"name": "毛血旺","price": 50,"flavor": "麻辣"},{"id": 5,"name": "菠菜拌粉丝","price": 22,"flavor": "五香"},{"id": 6,"name": "凉拌豆腐皮","price": 19,"flavor": "微辣"},{"id": 7,"name": "酱牛肉","price": 36,"flavor": "麻辣"},{"id": 8,"name": "鱼头豆腐汤","price": 32,"flavor": "五香"},{"id": 9,"name": "瘦肉鸡蛋白菜汤","price": 30,"flavor": "五香"},{"id": 10,"name": "西葫芦虾仁蒸饺","price": 26,"flavor": "五香"}]

则成功！

3、整合前端框架layui

这里是源代码：<https://github.com/southwind9801/orderingsystem>

前往git仓库下载layui文件，放入feign的static文件夹。

由于layui的一些特性，我们需要重新命名一个MenuVO类，进行数据封装

MenuVO.java

```java
package top.juntech.menu.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.util.List;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class MenuVO implements Serializable {
    private int code;
    private String msg;
    private int count;
    private List<Menu> data;
}
```

修改MenuController.java

```java
@GetMapping("/findall/{index}/{limit}")
public MenuVO findall(@PathVariable("index") int index,@PathVariable("limit") int limit){
    List<Menu> list = menuRepository.findAll(index,limit);
    return new MenuVO(0,"",menuRepository.count(),list);
}
```

修改feign：

```java
@GetMapping("/findall")
@ResponseBody
public MenuVO findall(@RequestParam("page") int page, @RequestParam("limit") int limit){
    int index = (page-1)*limit;
    return  menuFeign.findall(index,limit);
}

@GetMapping("/redirect/{location}")
public String redirect(@PathVariable("location") String location ){
    return location;
}
```



前端代码不需要做什么修改，先打开

<http://localhost:8030/client/findall?page=1&limit=10>

显示：

{"code": 0,"msg": "","count": 100,"data": [{"id": 2,"name": "烧椒扣肉","price": 46,"flavor": "微辣"},{"id": 3,"name": "栗子三杯鸡","price": 56,"flavor": "五香"},{"id": 4,"name": "毛血旺","price": 50,"flavor": "麻辣"},{"id": 5,"name": "菠菜拌粉丝","price": 22,"flavor": "五香"},{"id": 6,"name": "凉拌豆腐皮","price": 19,"flavor": "微辣"},{"id": 7,"name": "酱牛肉","price": 36,"flavor": "麻辣"},{"id": 8,"name": "鱼头豆腐汤","price": 32,"flavor": "五香"},{"id": 9,"name": "瘦肉鸡蛋白菜汤","price": 30,"flavor": "五香"},{"id": 10,"name": "西葫芦虾仁蒸饺","price": 26,"flavor": "五香"},{"id": 11,"name": "蛋炒饭","price": 18,"flavor": "五香"}]}

则成功！

新建Type类

```java
package top.juntech.menu.entity;

import lombok.Data;

import java.io.Serializable;

@Data
public class Type implements Serializable {
    private long id;
    private String name;
}
```

新建TypeRepository

```java
package top.juntech.menu.repository;

import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;
import org.springframework.web.bind.annotation.RequestParam;
import top.juntech.menu.entity.Type;

@Repository
public interface TypeRepository {

    @Select("select * from t_type where id = #{id}")
    public Type findById(@RequestParam("id")long id);
}
```

修改menuController

```java
@GetMapping("/findall/{index}/{limit}")
public MenuVO findall(@PathVariable("index") int index,@PathVariable("limit") int limit){
    List<Menu> list = menuRepository.findAll(index,limit);
    Iterator it = list.iterator();
    while (it.hasNext()){
        Menu menu = (Menu)it.next();
        Type type = typeRepository.findById(menu.getId());
        menu.setType(type);
    }
    return new MenuVO(0,"",menuRepository.count(),list);
}
```

重启服务提供者：打开<http://localhost:8020/menu/findall/0/10>

{"code": 0,"msg": "","count": 12,"data": [{"id": 1,"name": "香酥鸡","price": 39,"flavor": "五香","type": {"id": 1,"name": "热菜"}},{"id": 2,"name": "烧椒扣肉","price": 46,"flavor": "微辣","type": {"id": 2,"name": "凉菜"}},{"id": 3,"name": "栗子三杯鸡","price": 56,"flavor": "五香","type": {"id": 3,"name": "汤羹"}},{"id": 4,"name": "毛血旺","price": 50,"flavor": "麻辣","type": {"id": 4,"name": "主食"}},{"id": 5,"name": "菠菜拌粉丝","price": 22,"flavor": "五香","type": {"id": 5,"name": "烘焙"}},{"id": 6,"name": "凉拌豆腐皮","price": 19,"flavor": "微辣","type": null},{"id": 7,"name": "酱牛肉","price": 36,"flavor": "麻辣","type": null},{"id": 8,"name": "鱼头豆腐汤","price": 32,"flavor": "五香","type": null},{"id": 9,"name": "瘦肉鸡蛋白菜汤","price": 30,"flavor": "五香","type": null},{"id": 10,"name": "西葫芦虾仁蒸饺","price": 26,"flavor": "五香","type": null}]}

把Type,Menu实体类修改后，重启服务消费者:打开<http://localhost:8030/client/findall?page=1&limit=10>

显示：{"code": 0,"msg": "","count": 12,"data": [{"id": 1,"name": "香酥鸡","price": 39,"flavor": "五香","type": {"id": 1,"name": "热菜"}},{"id": 2,"name": "烧椒扣肉","price": 46,"flavor": "微辣","type": {"id": 2,"name": "凉菜"}},{"id": 3,"name": "栗子三杯鸡","price": 56,"flavor": "五香","type": {"id": 3,"name": "汤羹"}},{"id": 4,"name": "毛血旺","price": 50,"flavor": "麻辣","type": {"id": 4,"name": "主食"}},{"id": 5,"name": "菠菜拌粉丝","price": 22,"flavor": "五香","type": {"id": 5,"name": "烘焙"}},{"id": 6,"name": "凉拌豆腐皮","price": 19,"flavor": "微辣","type": null},{"id": 7,"name": "酱牛肉","price": 36,"flavor": "麻辣","type": null},{"id": 8,"name": "鱼头豆腐汤","price": 32,"flavor": "五香","type": null},{"id": 9,"name": "瘦肉鸡蛋白菜汤","price": 30,"flavor": "五香","type": null},{"id": 10,"name": "西葫芦虾仁蒸饺","price": 26,"flavor": "五香","type": null}]}



