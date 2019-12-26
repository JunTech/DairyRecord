# springboot与shiro

## 1.准备环境

### 1.1创建一个springboot工程

​	略

### 1.2 添加依赖及配置

pom.xml

```
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.shiro/shiro-spring-boot-starter -->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring-boot-starter</artifactId>
            <version>1.4.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.1.6.RELEASE</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.github.theborakompanioni/thymeleaf-extras-shiro -->
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
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
```

application.properties

```
server.port=8080
#shiro

shiro.enabled=true


```

这两个就够了

## 2.添加shiro自定义配置

MyshiroConfig.java

```
package top.juntech.springbootshiro.config;

import at.pollux.thymeleaf.shiro.dialect.ShiroDialect;
import org.apache.shiro.realm.Realm;
import org.apache.shiro.realm.text.TextConfigurationRealm;
import org.apache.shiro.spring.web.config.DefaultShiroFilterChainDefinition;
import org.apache.shiro.spring.web.config.ShiroFilterChainDefinition;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/13 9:51
 * @ClassName 类名
 * @Descripe 描述
 */
@Configuration
public class MyShiroConfig {

//自定义realm,添加了2个用户 user(snag/123) read 和 admin(admin/123) read and write权限
    @Bean
    public Realm realm() {
        TextConfigurationRealm realm = new TextConfigurationRealm();
        realm.setUserDefinitions("sang=123,user\n admin=123,admin");
        realm.setRoleDefinitions("admin=read,write\n user=read");
        return realm;
    }

    /*
    * ShiroFilterChainDefinition定义了过滤规则，login dologin 可以匿名访问logout注销 其余都需要授权
    *
    *
    * */
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition(){
        DefaultShiroFilterChainDefinition defaultShiroFilterChainDefinition =
                new DefaultShiroFilterChainDefinition();
        defaultShiroFilterChainDefinition.addPathDefinition("/login", "anon");
        defaultShiroFilterChainDefinition.addPathDefinition("/dologin", "anon");
        defaultShiroFilterChainDefinition.addPathDefinition("/logout", "logout");
        defaultShiroFilterChainDefinition.addPathDefinition("/**", "authc");
        return defaultShiroFilterChainDefinition;
    }
	//这个是因为我这里使用了thymeleaf，要在themeleaf中使用shiro标签才引入这个bean
    @Bean
    public ShiroDialect shiroDialect(){
        return new ShiroDialect();
    }
}

```

## 3.添加控制器

UserController.java

```
package top.juntech.springbootshiro.controller;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.annotation.Logical;
import org.apache.shiro.authz.annotation.RequiresRoles;
import org.apache.shiro.subject.Subject;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/13 10:01
 * @ClassName 类名
 * @Descripe 描述
 */
@Controller
public class UserController {


    /*
    *在dologin方法中，首先构造一个usernamepasswordtoken实例然后获取一个subject对象并调用这个对象的		*login方法
    *登录成功--》index 登录失败 --》报异常---》返回登录页面
    * */
    @PostMapping("/dologin")
    public String dologin(String username, String password, Model model){
        UsernamePasswordToken token = new UsernamePasswordToken(username,password);
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(token);
        }catch (AuthenticationException e){
            model.addAttribute("error","用户名密码错误！");
            return "login";
        }

        return "redirect:/index";
    }
	
	//进入admin，需要admin权限
    @RequiresRoles("admin")
    @GetMapping("/admin")
    public String admin(){
        return "admin";
    }
	//进入user,需要user 或admin权限
    @RequiresRoles(value = {"admin","user"},logical = Logical.OR )
    @GetMapping("/user")
    public String user(){
        return "user";
    }
}

```

在这里，我们定义了一个属性error,在全局异常处理那里会用到

为了方便，我们定义一个mvc配置类，配置一些不需要授权就能访问的页面

MyWebMvcConfig.java

```
package top.juntech.springbootshiro.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/13 10:17
 * @ClassName 类名
 * @Descripe 描述
 */
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    /*
    * 对于其他不需要角色就能访问的接口，直接在mvc配置里配置
    *
    * */
    @Override
   public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login").setViewName("login");
        registry.addViewController("/index").setViewName("index");
        registry.addViewController("/unauthorized").setViewName("unauthorized");

    }
}

```

但是这里/unauthorized我们没有，怎么办呢，我们在定义一个全局的异常处理,处理授权异常

```
package top.juntech.springbootshiro.controller;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;

import javax.naming.AuthenticationException;

/**
 * @author juntech
 * @version ${version}
 * @date 2019/12/13 10:26
 * @ClassName 类名
 * @Descripe 全局异常处理类
 */
@ControllerAdvice
public class ExceptionController {

    @ExceptionHandler(AuthenticationException.class)
    public ModelAndView error(AuthenticationException e){
        ModelAndView mv = new ModelAndView();
        mv.addObject("error",e.getMessage());
        return mv;
    }
}

```

当用户访问未授权的资源时，跳转到unauthorized视图中并携带error信息