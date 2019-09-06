# springcloud教程第4篇：Zuul---网关

在微服务架构中，需要几个基础的服务治理组件，包括服务注册与发现、服务消费、负载均衡、断路器、智能路由、配置管理等，由这几个基础组件相互协作，共同组建了一个简单的微服务系统。一个简易的微服务系统如下图：

![Azure (1).png](https://forezp.obs.myhuaweicloud.com/img/jianshu/2279594-6b7c148110ebc56e.png)

在Spring Cloud微服务系统中，一种常见的负载均衡方式是，客户端的请求首先经过**负载均衡（zuul、Ngnix）**，再到达服务网关（zuul集群），然后再到具体的服务，服务统一注册到高可用的服务注册中心集群，服务的所有的配置文件由配置中心管理，配置服务的配置文件放在git仓库或本地，方便开发人员随时改配置。

## 1、什么是Zuul

Zuul的主要功能是路由转发和过滤器。路由功能是微服务的一部分，比如／api/user转发到到user服务，/api/shop转发到到shop服务。zuul默认和Ribbon结合实现了负载均衡的功能。

zuul有以下功能：

- Authentication
- Insights
- Stress Testing
- Canary Testing
- Dynamic Routing
- Service Migration
- Load Shedding
- Security
- Static Response handling
- Active/Active traffic management

## 2、准备工作

继续使用上一节的工程

## 3、创建zuul工程

pom.xml:需要引入zuul组件

```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```

main程序：加入@EnableZuulProxy注解，开启zuul功能

```java
@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class ServiceZuulApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceZuulApplication.class, args);
	}
}
```

application.yml:

```yml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8769
spring:
  application:
    name: service-zuul
zuul:
  routes:
  #将service-id为service-ribbon转发到api-a接口
    api-a:
      path: /api-a/**
      serviceId: service-ribbon
#将service-id为service-feign转发到api-b接口
    api-b:
      path: /api-b/**
      serviceId: service-feign
```

依次运行这五个工程;打开浏览器访问：http://localhost:8769/api-a/hi?name=jd ;浏览器显示：

> hi jd,i am from port:8762

打开浏览器访问：http://localhost:8769/api-b/hi?name=jd ;浏览器显示：

> hi jd,i am from port:8762

这说明zuul起到了路由转发的作用

## 4、服务过滤

zuul不仅只是路由，并且还能过滤，做一些安全验证。继续改造工程；

```
@Component
public class MyFilter extends ZuulFilter{

    private static Logger log = LoggerFactory.getLogger(MyFilter.class);
    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s >>> %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("token");
        if(accessToken == null) {
            log.warn("token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            try {
                ctx.getResponse().getWriter().write("token is empty");
            }catch (Exception e){}

            return null;
        }
        log.info("ok");
        return null;
    }
}
```

- filterType：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：
  - pre：路由之前
  - routing：路由之时
  - post： 路由之后
  - error：发送错误调用
- filterOrder：过滤的顺序
- shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
- run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。

这时访问：http://localhost:8769/api-a/hi?name=jd ；网页显示：

> token is empty

访问 http://localhost:8769/api-a/hi?name=jd&token=22 ； 网页显示：

> hi jd,i am from port:8762

## 5、参考资料

- [zuul](https://projects.spring.io/spring-cloud/spring-cloud.html#_router_and_filter_zuul)

  ​