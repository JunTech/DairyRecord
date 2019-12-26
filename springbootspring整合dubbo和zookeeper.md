# springboot/spring整合dubbo和zookeeper

## 1.安装zookeeper

### 1.1下载zookeeper

地址：https://archive.apache.org/dist/zookeeper/

- 下载后解压
- 初次点击zkServer.cmd启动会报错、闪退现象
- 如出现闪退无法看到错误日志，用编辑器打开zkServer.cmd文件在最底下添加pause在启动就能看到错误日志
- 因为没有zoo.cf文件，需要前往conf目录下复制zoo_sample.cfg文件，修改文件名为zoo.cfg

这下面是我的配置，可以参考一下：

```
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=D:\devtools\zookeeper-3.4.14\data
dataLogDir=D:\devtools\zookeeper-3.4.14\log
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

```

### 1.2启动：

点击bin目录下的zkServer.cmd启动，前提是安装好了java环境

### 1.3使用zkCli.cmd测试

- ls /：列出zookeeper根下保存的所有节点
- create –e /juntech 123：创建一个atguigu节点，值为123
- get /juntech：获取/juntech节点的值

## 2.安装dubbo-admin可视化界面

这里我直接将我打包好的dubbo-admin献上, [点击下载](/download/dubbo-admin-0.0.1-SNAPSHOT.jar")

下载之后进入包里面将，dubbo-admin-0.0.1-SNAPSHOT.jar\BOOT-INF\classes目录下的applciaiton.properties文件的dubbo.registry.address改为

```
dubbo.registry.address=zookeeper://127.0.0.1:2181
```

### 2.1运行

```
java -jar dubbo-admin-0.0.1-SNAPSHOT.jar
```

打开lcoalhost:7001，用户名密码都是root，就可以对dubbo服务的服务提供者，服务消费者进行监控了。

如下图：

![](https://img.vim-cn.com/0b/003cfa2d2ae13dd0186be5bcee6ef72bf88fde.png)

## 3.准备工作

工具：idea,jdk8,dubbo-admin,zookeeper,lombok

项目结构：![](https://img.vim-cn.com/8c/7e481b9e4d184c735babc5af8a1bd2147b0382.png)

dubbo架构：![](http://dubbo.apache.org/img/architecture.png)

### 3.1开始整合

将公共接口和bean文件放在common-api包下，实现还是在原来的包里面

首先我们在common-api下建一个bean文件：

```
package top.juntech.commonapi.bean;

import lombok.*;

import java.io.Serializable;

@Setter
@Getter
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class UserAddress implements Serializable {
    private Integer id;
    private String userAddress; //用户地址
    private String userId; //用户id
    private String consignee; //收货人
    private String phoneNum; //电话号码
    private String isDefault; //是否为默认地址    Y-是     N-否
}

```

有2个服务：userservice,orderservice

1.服务消费者服务

```
package top.juntech.commonapi.service;

import top.juntech.commonapi.bean.UserAddress;

import java.util.List;

public interface OrderService {
    //初始化订单
    public List<UserAddress> initOrder(String userId);
}

```

2.服务提供者服务

```
package top.juntech.commonapi.service;

import top.juntech.commonapi.bean.UserAddress;

import java.util.List;

public interface UserService {
    List<UserAddress> getUserAddressList(String userId);
}
```

pom文件：

引入一个lombok就行了

```
<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
```

### 3.2 服务提供者 userservice

spring配置方式：

#### 3.2.1建立一个service-provider包

包下面建立一个实现userservice的实现类：userserviceimpl,

在这里我们就不连接数据库了，直接自己早一点数据来测试就行了

```
package top.juntech.serviceprovider.service.impl;

import org.apache.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;
import top.juntech.commonapi.bean.UserAddress;
import top.juntech.commonapi.service.UserService;

import java.util.Arrays;
import java.util.List;

@Service
@Component
public class UserServiceImpl implements UserService {
    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        //模拟获取数据过程，这里为简化，自定义两个地址对象返回
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座9层", "1", "王老师", "010-56253825", "N");
        return Arrays.asList(address1,address2);
    }
}

```



#### 3.2.2 建立xml文件来配置dubbo暴露服务

provider.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
    http://dubbo.apache.org/schema/dubbo
    http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 1、指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名） -->
    <dubbo:application name="user-provider"></dubbo:application>

    <!-- 2、指定注册中心的位置  -->
    <!-- <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry> -->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>

    <!-- 3、指定通信规则（通信协议   通信端口） -->
    <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>

    <!-- 4、暴露服务   ref：指向服务的真正的实现对象 -->
    <dubbo:service interface="top.juntech.commonapi.service.UserService" ref="userServiceImpl"></dubbo:service>

    <!-- 服务的实现对象 -->
    <bean id="userServiceImpl" class="top.juntech.serviceprovider.service.impl.UserServiceImpl"></bean>

</beans>
```

#### 3.2.3 pom.xml文件添加要导入的依赖

在这里我们需要 dubbo ,zkclient,以及curator,curator-recipes..,同时还要导入common-api的pom坐标

```
      <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
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
        <!--dubbo-->
        <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.4.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>

        <!--zookeeper 日志排除-->
        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.7</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>top.juntech</groupId>
            <artifactId>common-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```

#### 3.2.4 启动类

```
package top.juntech.serviceprovider;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ServiceProviderApplication {

    public static void main(String[] args)  throws Exception{
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("provider.xml");
        context.start();
        System.in.read();
    }

}


```



### 3.3服务消费者 orderservice

实现类：

```
package top.juntech.serviceconsumer.service.impl;

import org.apache.dubbo.config.annotation.Reference;
import org.apache.dubbo.config.annotation.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import top.juntech.commonapi.bean.UserAddress;
import top.juntech.commonapi.service.OrderService;
import top.juntech.commonapi.service.UserService;

import java.util.List;

@Service
@Component
public class OrderServiceImpl implements OrderService {

    @Autowired
    private UserService userService;
    @Override
    public List<UserAddress> initOrder(String userId) {
        System.out.println("用户id："+userId);

        List<UserAddress> addressList = userService.getUserAddressList(userId);
        for (UserAddress userAddress : addressList) {
            System.out.println(userAddress.getUserAddress());
        }
        return addressList;
    }
}

```

consumer.xml

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
        http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd
        http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <!-- 开启包扫描 -->
    <context:component-scan base-package="top.juntech.serviceconsumer.service.impl"></context:component-scan>

    <!-- 1、指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名） -->
    <dubbo:application name="order-consumer"></dubbo:application>
    <!-- 2、指定注册中心的位置 -->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>
    <!-- 3、声明需要调用的远程服务的接口；生成远程服务代理 -->
    <dubbo:reference id="userService" interface="top.juntech.commonapi.service.UserService"></dubbo:reference>
</beans>
```

pom.xml 与服务提供者的基本一致，只需要添加 spring-web

```
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
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
        <!--dubbo-->
        <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.4.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.github.sgroschupf/zkclient -->
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>

        <!--zookeeper 日志排除-->
        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-framework -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.7</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>top.juntech</groupId>
            <artifactId>service-provider</artifactId>
            <version>1.0.1</version>
        </dependency>
        <dependency>
            <groupId>top.juntech</groupId>
            <artifactId>common-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

启动类：

```
package top.juntech.serviceconsumer;


import org.springframework.context.support.ClassPathXmlApplicationContext;
import top.juntech.commonapi.service.OrderService;

public class ServiceConsumerApplication {

    public static void main(String[] args) throws Exception{
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("consumer.xml");
        OrderService orderService = context.getBean(OrderService.class);
        orderService.initOrder("1");
        System.out.println("调用完成....");
        System.in.read();
    }

}

```

### 3.4springboot方式整合dubbo

创建2个springboot工程

如下：![](https://img.vim-cn.com/29/ba595f47b550126f1f0059a0958eb797afa906.png)

过程和spring整合dubbo差不多，在这里没有使用注解的方式，还是使用的xml方式配置dubbo,注解方式老是出错，显示地址被占用，很坑

#### 3.4.1服务提供者

```
package top.juntech.springbootserviceprovider.service.impl;

import org.apache.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;
import top.juntech.commonapi.bean.UserAddress;
import top.juntech.commonapi.service.UserService;

import java.util.Arrays;
import java.util.List;

@Service
@Component
public class UserServiceImpl implements UserService {
    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        //模拟获取数据过程，这里为简化，自定义两个地址对象返回
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座9层", "1", "王老师", "010-56253825", "N");
        return Arrays.asList(address1,address2);
    }
}

```

启动类

```
package top.juntech.springbootserviceprovider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ImportResource;

@SpringBootApplication
//@EnableDubbo
@ImportResource(locations = "classpath:provider.xml")
public class SpringbootServiceProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootServiceProviderApplication.class, args);
    }

}

```

provider.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
    http://dubbo.apache.org/schema/dubbo
    http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!-- 1、指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名） -->
    <dubbo:application name="user-provider"></dubbo:application>

    <!-- 2、指定注册中心的位置  -->
    <!-- <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry> -->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>

    <!-- 3、指定通信规则（通信协议   通信端口） -->
    <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>

    <!-- 4、暴露服务   ref：指向服务的真正的实现对象 -->
    <dubbo:service interface="top.juntech.commonapi.service.UserService" ref="userServiceImpl"></dubbo:service>

    <!-- 服务的实现对象 -->
    <bean id="userServiceImpl" class="top.juntech.springbootserviceprovider.service.impl.UserServiceImpl"></bean>

</beans>
```

pom文件与之前的一样就不在重复了

#### 3.4.2 服务消费者

```
package top.juntech.springbootserviceconsumer.service.impl;

import org.apache.dubbo.config.annotation.Reference;
import org.apache.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;
import top.juntech.commonapi.bean.UserAddress;
import top.juntech.commonapi.service.OrderService;
import top.juntech.commonapi.service.UserService;

import java.util.List;

@Component
//这里是dubbo的注解service,作用是暴露服务
@Service
public class OrderServiceImpl implements OrderService {
	//显示被引用的服务
    @Reference
    private UserService userService;

    @Override
    public List<UserAddress> initOrder(String userId) {
        return userService.getUserAddressList(userId);
    }
}

```

启动类：

```
package top.juntech.springbootserviceconsumer;

import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ImportResource;

@SpringBootApplication
//@EnableDubbo
@ImportResource(locations = "classpath:consumer.xml")
public class SpringbootServiceConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootServiceConsumerApplication.class, args);
    }

}

```

方便测试，写了一个controller：

```
package top.juntech.springbootserviceconsumer.controller;

import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import top.juntech.commonapi.bean.UserAddress;
import top.juntech.commonapi.service.OrderService;
import top.juntech.commonapi.service.UserService;

import java.util.List;

@RestController
@RequestMapping("/consumer")
public class ConsumerController {

    @Autowired
    private OrderService orderService;

    @ResponseBody
    @RequestMapping("/initOrder")
    public List<UserAddress> initOrder(@RequestParam("uid") String userId){

        return orderService.initOrder(userId);
    }
}

```

启动服务提供者，服务消费者，前提是先启动了zookeeper

打开可视化界面观察服务：

127.0.0.1:7001

界面如下：

![](https://img.vim-cn.com/38/3946004a3edabd2d5ed94e98c7a94a4fd2702b.png)

至此，spring/springboot整合dubbo就结束了。