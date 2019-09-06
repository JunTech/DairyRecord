# Springboot整合Spring Data Jpa

## 1、引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.1.7.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!--<scope>runtime</scope>-->
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
```

## 2、编写配置文件

```properties
#mysql
spring.datasource.password=123456
spring.datasource.username=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url= jdbc:mysql://127.0.0.1:13306/crawler?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC

server.port= 8888

#logging
# To set logs level as per your need.
logging.level.org.jpa = debug
logging.level.top.juntech.autohome = trace

#data jpa
spring.jpa.show-sql=true
spring.jpa.generate-ddl=true
spring.jpa.database=mysql
#有create,update...，我这里使用update,可以在没有表的时候直接创建表，并且后续实体类更新，表也会跟着更新
spring.jpa.hibernate.ddl-auto=update
#设置hinernate方言为mysql
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```

## 3、编写实体类

```java
package top.juntech.autohome.entity;

import lombok.Data;

import javax.persistence.*;
import java.io.Serializable;
import java.util.Date;

@Data
@Entity
@Table(name = "car_test")
public class Car_Test  implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(length = 150)
    private String title;
    @Column(length = 150)
    private Integer test_speed;
    @Column(length = 150)
    private Integer test_oil;
    @Column(length = 20)
    private String editor_name1;
    @Column(length = 20)
    private String editor_name2;
    @Column(length = 20)
    private String editor_name3;
    @Column(length = 1000)
    private String editor_comment1;
    @Column(length = 1000)
    private String editor_comment2;
    @Column(length = 1000)
    private String editor_comment3;
    @Column(length = 1000)
    private String image;
    private Date created;
    private Date updated;

}
```

## 4、运行程序

打开navicat,查看数据库，看是否生成了实体类表car_test.

到此，就成功了！