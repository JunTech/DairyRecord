---
title: Shiro、Spring Security
author: Juntech
top: true
cover: true
toc: true
mathjax: false
copyright: true
summary: Shiro、Spring Security
categories:
  - ShiroSpring
  - Security
tags:
  - ShiroSpring
  - Security
keywords: Shiro、Spring Security
abbrlink: 1431788a
date: 2019-09-11 16:30:20
---
# Shiro、Spring Security
## Spring Boot 整合 Spring Security
## 快速上⼿ Spring Security
###1、创建 Maven ⼯程，pom.xml
```
<parent> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-parent</artifactId> <version>2.1.5.RELEASE</version>
</parent> <dependencies> <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-web</artifactId>
</dependency> <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency> <dependency> <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-security</artifactId>
</dependency>
</dependencies> 
```
### 2、创建 Handler
```
package com.southwind.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
@Controller
public class HelloHandler {
@GetMapping("/index")
public String index(){
return "index";
 }
}
```
### 3、创建 HTML
```
<!DOCTYPE html>
<html lang="en"> <head><meta charset="UTF-8"> <title>Title</title>
</head> <body><h1>Hello World</h1>
</body>
</html> 4、创建 application.yml
spring:
 thymeleaf:
 prefix: classpath:/templates/
 suffix: .html
 ```
### 5、创建启动类 Application
```
package com.southwind.controller;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class Application {
public static void main(String[] args) {
SpringApplication.run(Application.class,args);
 }
}
```
### 6、设置⾃定义密码
```
spring:
 thymeleaf:
 prefix: classpath:/templates/
 suffix: .html
 security:
 user:
 name: admin
 password: 123123
 ```
权限管理
定义两个 HTML 资源：index.html、admin.html，同时定义两个⻆⾊ ADMIN 和 USER，ADMIN 拥有
访问 index.html 和 admin.html 的权限，USER 只有访问 index.html 的权限。
### 7、创建 SecurityConfig 类。
```
package com.southwind.config;
import org.springframework.context.annotation.Configuration;
import
org.springframework.security.config.annotation.authentication.builders.Authent
icationManagerBuilder;
import
org.springframework.security.config.annotation.web.builders.HttpSecurity;
import
org.springframework.security.config.annotation.web.configuration.EnableWebSecu
rity;
import
org.springframework.security.config.annotation.web.configuration.WebSecurityCo
nfigurerAdapter;
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
@Override
protected void configure(AuthenticationManagerBuilder auth) throws
Exception {
auth.inMemoryAuthentication().passwordEncoder(new MyPasswordEncoder())
 .withUser("user").password(new
MyPasswordEncoder().encode("000")).roles("USER")
 .and()
 .withUser("admin").password(new
MyPasswordEncoder().encode("123")).roles("ADMIN","USER");
 }
@Override
protected void configure(HttpSecurity http) throws Exception {
http.authorizeRequests().antMatchers("/admin").hasRole("ADMIN")
 .antMatchers("/index").access("hasRole('ADMIN') or
hasRole('USER')")
 .anyRequest().authenticated()
 .and()
 .formLogin()
 .loginPage("/login")
 .permitAll()
 .and()
 .logout()
 .permitAll()
 .and()
 .csrf()
 .disable();
 }
}

```
### 8、修改 Handler
```
package com.southwind.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
@Controller
public class HelloHandler {
@GetMapping("/index")
public String index(){
return "index";
 }
@GetMapping("/admin")
public String admin(){
return "admin";
 }
@GetMapping("/login")
public String login(){
return "login";
 }
} 
```
### 9、login.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"> <html lang="en"> <head><meta charset="UTF-8"> <title>Title</title>
</head> <body><form th:action="@{/login}" method="post">
⽤户名：<input type="text" name="username"/><br/>
密码：<input type="text" name="password"/><br/>
<input type="submit" value="登录"/>
</form>
</body>
</html>
10、index.html
<!DOCTYPE html>
<html lang="en"> <head><meta charset="UTF-8"> <title>Title</title>
</head> <body><h1>Hello World</h1> <form action="/logout" method="post"> <input type="submit" value="退出"/>
</form>
</body>
</html>
11、admin.html
<!DOCTYPE html>
<html lang="en"> <head><meta charset="UTF-8"> <title>Title</title>
</head> <body><h1>后台管理系统</h1> <form action="/logout" method="post"> <input type="submit" value="退出"/>
</form>
</body>
</html>
```