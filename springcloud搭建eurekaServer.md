# springcloud 搭建 eurekaService

## 	1、初始化项目

​	首先创建一个spring init项目,然后在选择依赖库时我们只需要eureka Server,所以只选择eureka server就行了，打开pom.xml:

​	jdk8大概显示为：

​		

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
    	<relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.juntech</groupId>
    <artifactId>microservice</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>microservice</name>
    <description>Demo project for Spring Boot</description>
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

jdk9缺少JAXB API

```xml
<!-- 解决 JDK 9 以上没有 JAXB API 的问题 -->
<dependency>
<groupId>javax.xml.bind</groupId>
<artifactId>jaxb-api</artifactId>
<version>2.3.0</version>
</dependency>
<dependency>
<groupId>com.sun.xml.bind</groupId>
<artifactId>jaxb-impl</artifactId>
<version>2.3.0</version>
</dependency>
<dependency>
<groupId>com.sun.xml.bind</groupId>
<artifactId>jaxb-core</artifactId>
<version>2.3.0</version>
</dependency>
<dependency>
<groupId>javax.activation</groupId>
<artifactId>activation</artifactId>
<version>1.1.1</version>
</dependency>
</dependencies>
<dependencyManagement>
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-dependencies</artifactId>
<version>Finchley.SR2</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>
```

## 2、创建配置文件application.yml

```yml
server:
	port: 8761
eureka:
	client:
		register-with-eureka: false
		fetch-registry: false
		service-url:
			defaultZone: http://localhost:8761/eureka/
```



一些属性的说明：

1. ​	server.port ：当前 Eureka Server 服务端⼝口。
2. ​	eureka.client.register-with-eureka ：是否将当前的 Eureka Server 服务作为客户端进⾏行行注册。
3. ​	eureka.client.fetch-fegistry ：是否获取其他 Eureka Server 服务的数据。
4. ​	eureka.client.service-url.defaultZone ：注册中⼼心的访问地址。



## 3、创建启动类

```java
package top.juntech.microservice;
import org.springframework.boot.SpringApplication;

import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication

@EnableEurekaServer    //要开启eureka服务必须在主运行程序上加上该注解

public class EurekaServerApplication {

public static void main(String[] args) {

	SpringApplication.run(EurekaServerApplication.class,args);

}

}
```

**注解说明：**
@SpringBootApplication ：声明该类是 Spring Boot 服务的⼊入⼝口。
@EnableEurekaServer ：声明该类是⼀一个 Eureka Server 微服务，提供服务注册和服务发现功能，即
注册中⼼心。

## 4、Eureka Client 代码实现

1、创建module，引入eurekadiscoveryclient，starter

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2、创建application.yml

```yml
server:
  port: 8010
spring:
  application:
  name: provider
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

**其中属性说明：**

```
spring.application.name ：当前服务注册在 Eureka Server 上的名称。
eureka.client.service-url.defaultZone ：注册中⼼心的访问地址。
eureka.instance.prefer-ip-address ：是否将当前服务的 IP 注册到 Eureka Server。
```

**创建启动类**

```java
package top.juntech.springcloud.eurekaclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class EurekaclientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaclientApplication.class, args);
    }

}
```

**其中属性说明**

@EnableDiscoveryClient : 注解开启eureka client服务



## 5、创建实体类

需要用到lombok，idea如何安装lombok，请转到https://jingyan.baidu.com/article/0a52e3f4e53ca1bf63ed725c.html

Student类

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student implements Serializable {

    private long id;
    private String name;
    private int age;
}
```

dao层：

```java
package top.juntech.springcloud.eurekaclient.dao;

import org.springframework.stereotype.Repository;
import top.juntech.springcloud.eurekaclient.entity.Student;

import java.util.Collection;

@Repository
public interface StudentDao {
    public Collection<Student> findAll();
    public Student findById(long id);
    public void saveOrUpdate(Student student);
    public void deleteById(long id);
}
```

dao.impl层

```java
package top.juntech.springcloud.eurekaclient.dao.impl;

import top.juntech.springcloud.eurekaclient.dao.StudentDao;
import top.juntech.springcloud.eurekaclient.entity.Student;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

public class StudentDaoImpl implements StudentDao {
    private static Map<Long,Student> studentMap;

    static {
        studentMap = new HashMap<>();
        studentMap.put(1L, new Student(1L, "张三", 22));
        studentMap.put(2L, new Student(2L, "李李四", 23));
        studentMap.put(3L,new Student(3L,"王五",24));
    }

    @Override
    public Collection<Student> findAll() {
        return studentMap.values();
    }
    @Override
    public Student findById(long id) {
        return studentMap.get(id);
    }
    @Override
    public void saveOrUpdate(Student student) {
        studentMap.put(student.getId(),student);
    }
    @Override
    public void deleteById(long id) {
        studentMap.remove(id);
    }


}
```

## 6、创建controller

```java
package top.juntech.springcloud.eurekaclient.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import top.juntech.springcloud.eurekaclient.dao.StudentDao;
import top.juntech.springcloud.eurekaclient.entity.Student;

import java.util.Collection;

@RestController
@RequestMapping("/student")
public class StudentController {
    @Autowired
    private StudentDao studentRepository;

    @GetMapping("/findAll")
    public Collection<Student> findAll(){
        return studentRepository.findAll();
    }
    @GetMapping("/findById/{id}")
    public Student findById(@PathVariable("id") long id){
        return studentRepository.findById(id);
    }
    @PostMapping("/save")
    public void save(@RequestBody Student student){
        studentRepository.saveOrUpdate(student);
    }
    @PutMapping("/update")
    public void update(@RequestBody Student student){
        studentRepository.saveOrUpdate(student);
    }
    @DeleteMapping("/deleteById/{id}")
    public void deleteById(@PathVariable("id") long id){
        studentRepository.deleteById(id);
    }
}
```

