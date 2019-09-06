# mabtis整合踩坑记录

## 1、The server time zone value '**' is unrecognized or represents more than one time zone

### 错误环境：

**mysql版本：5.6.0**

### 错误提示:

The server time zone value “乱码” is unrecognized or represents more than one time zone

### 解决方案：

- 方案1、在项目代码-数据库连接URL后，加上 （注意大小写必须一致） 
  jdbc.url=jdbc:mysql://127.0.0.1:3306/**?serverTimezone=UTC
- 方案2、在mysql中设置时区，默认为SYSTEM 
  set global time_zone=’+8:00’



## 2、java.sql.SQLException: Access denied for user ''@'localhost' (using password: NO)

### 错误环境：

**mysql版本：5.6.0**

### **错误提示:**

java.sql.SQLException: Access denied for user ''@'localhost' (using password: NO)

### 解决方案：

因为此次采坑是使用的yml,就采用yml的写法：

**第一步**：检查url写错没有

**第二步**：检查用户名密码是否错误

**第三步**：检查用户名密码是不是写的data-username,data-password,如果是，把data-去掉，就可以了



**yml----》datasource配置**

```
datasource:
  driver-class-name: com.mysql.jdbc.Driver
  url: jdbc:mysql://127.0.0.1:3306（你的数据库端口，默认是3306）/数据库名字?useUnicode=true&amp;characterEncoding=UTF-8
  username: root
  password: 1006（你的密码）
```

以上用的是application.yml配置文件

**记住：username和password 前面不能加 data-**



## 3、mysql maven引入

```yaml
<!--mysql-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
    <!--<version>5.6.0</version>-->
</dependency>
```

低的版本可以引入高的，但高的版本不能引入低的版本的mysql