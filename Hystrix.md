# springcloud教程第3篇：Hystrix----熔断器

为了保证服务的高可用性，一般将单个服务进行集群部署，但由于自身原因或网络原因，服务不能保证100%能用，如果单个服务出现问题，会造成线程阻塞，极易导致服务崩溃，造成雪崩效应。

为了解决这个问题，提出了熔断器概念----Hystrix!

## 一、断路器简介

Netflix开源了Hystrix组件，实现了断路器模式，SpringCloud对这一组件进行了整合。 在微服务架构中，一个请求需要调用多个服务是非常常见的，如下图：

![HystrixGraph.png](https://forezp.obs.myhuaweicloud.com/img/jianshu/2279594-08d8d524c312c27d.png)

较底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。

![HystrixFallback.png](https://forezp.obs.myhuaweicloud.com/img/jianshu/2279594-8dcb1f208d62046f.png)

断路打开后，可用避免连锁故障，fallback方法可以直接返回一个固定值。

## 二、准备工作

这篇文章基于服务消费者文章的工程，首先启动工程，启动eureka-server 工程；启动service-hi工程，它的端口为8762。

## 三、在ribbon使用断路器

改造serice-ribbon 工程的代码，首先在pox.xml文件中加入spring-cloud-starter-hystrix的起步依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

在程序的启动类ServiceRibbonApplication 加@EnableHystrix注解开启Hystrix：

```java
@SpringBootApplication
@EnableDiscoveryClient
//开启熔断规则
@EnableHystrix
public class ServiceRibbonApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceRibbonApplication.class, args);
	}

	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

}
```

改造HelloService类，在hiService方法上加上@HystrixCommand注解。

```java
@Service
public class HelloService {
@Autowired
RestTemplate restTemplate;

@HystrixCommand(fallbackMethod = "hiError")
public String hiService(String name) {
    return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
}

public String hiError(String name) {
    return "hi,"+name+",sorry,error!";
}
}
```
如果发生熔断就会返回hiError方法，即显示："hi,"+name+",sorry,error!";

如果正常则继续使用hiService(String name)方法，显示：restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);获得的json数据

启动：service-ribbon 工程，当我们访问http://localhost:8764/hi?name=jd,浏览器显示：

> hi,jd,i am from port:8762

此时关闭 service-hi 工程，当我们再访问http://localhost:8764/hi?name=jd，浏览器会显示：

> hi ,jd,orry,error!(这是由于网络原因，服务已关闭，网络不可达，)

这就说明当 service-hi 工程不可用的时候，service-ribbon调用 service-hi的API接口时，会执行快速失败，直接返回一组字符串，而不是等待响应超时，这很好的控制了容器的线程阻塞。

接下来，在Feign中使用熔断器

## 四、Feign中使用断路器

Feign是自带断路器的，在D版本的Spring Cloud中，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：

> feign.hystrix.enabled=true

基于service-feign工程进行改造，只需要在FeignClient的SchedualServiceHi接口的注解中加上fallback的指定类就行了，代码如下：

```java
@FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

参数解析：

​	**value = "service-hi"：在注册中心的要使用的服务名**	

​	**fallback = SchedualServiceHiHystric.class：只要发生熔断，就返回该类里面对应的方法**

SchedualServiceHiHystric需要实现SchedualServiceHi 接口，并注入到Ioc容器中，代码如下：

```java
@Component
//需要注册为一个组件
public class SchedualServiceHiHystric implements SchedualServiceHi {
    @Override
    public String sayHiFromClientOne(String name) {
        return "sorry "+name;
    }
}
```

启动service-feign工程，浏览器打开http://localhost:8765/hi?name=jd,注意此时service-hi工程没有启动，网页显示：

> sorry jd

打开service-hi工程，再次访问，浏览器显示：

> hi jd,i am from port:8762

这证明断路器起到作用了。

## 五、Hystrix Dashboard (断路器：Hystrix 仪表盘)

基于service-ribbon 改造，Feign的改造和这一样。

首选在pom.xml引入spring-cloud-starter-hystrix-dashboard的起步依赖：

```xml
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
	</dependency>
```
在主程序启动类中加入@EnableHystrixDashboard注解，开启hystrixDashboard：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
@EnableHystrixDashboard
public class ServiceRibbonApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceRibbonApplication.class, args);
	}

	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

}
```

打开浏览器：访问http://localhost:8764/hystrix,界面如下

![Paste_Image.png](https://forezp.obs.myhuaweicloud.com/img/jianshu/2279594-64f5fa9d0d96ee21.png)

点击monitor stream，进入下一个界面，访问：http://localhost:8764/hi?name=jd

此时会出现监控界面：

![Paste_Image.png](https://forezp.obs.myhuaweicloud.com/img/jianshu/2279594-755cd7ce5c066649.png)

六、参考资料

- [feign-hystrix](https://projects.spring.io/spring-cloud/spring-cloud.html#spring-cloud-feign-hystrix)
- [hystrix-dashboard](https://projects.spring.io/spring-cloud/spring-cloud.html#_circuit_breaker_hystrix_dashboard)

