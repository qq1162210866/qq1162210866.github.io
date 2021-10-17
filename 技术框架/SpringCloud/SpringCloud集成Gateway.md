## SpringCloud集成Gateway

​	拖了很久，目前最后一篇SpringCloud的博客终于写了。这个坑算是告一段落了。最后一篇博客就是整合Gateway。也不说其他的了，直接开始吧。

​	其实写了这么久，应该了解了SpringCloud是有多个组件组成的，每个部分都有许多替代品，网关这个部分也不例外。其实最开始我一直在纠结选择哪个组件。目前比较好和流行的有Zuul和Gateway。但是Zuul2.0后就闭源了，所以一直在纠结，最后还是选择了Gateway。

​	Gateway的官网地址：[https://spring.io/projects/spring-cloud-gateway](https://spring.io/projects/spring-cloud-gateway)其实这个地址其他的组件也能找到，之前没有关注这个网址，后续大家可以在上面了解一下详细的写法和demo。

​	该项目提供了一个用于在Spring MVC之上构建API网关的库。Spring Cloud Gateway旨在提供一种简单而有效的方法来路由到API，并为它们提供跨领域的关注，例如：安全性，监视/指标和弹性。它的功能有以下几点。

- 基于Spring Framework 5，Project Reactor和Spring Boot 2.0构建

- 能够匹配任何请求属性上的路由。

- 谓词和过滤器特定于路由。

- Hystrix断路器集成。

- Spring Cloud DiscoveryClient集成

- 易于编写的谓词和过滤器

- 请求速率限制

- 路径改写

​	不难看到其实网关的一部分功能和其他的组件有些重合。这个后面会有一个问题。关于Gateway的原理这里不再叙述了，作为后续的一个扩展点（又是一个坑）。就直接整合。

​	需要添加Gateway的依赖，同时Gateway作为一个服务也是需要注册到Eureka注册中心的。

​	使用IDEA直接创建一个新模块，指定好名称和需要添加的依赖，这里只选择Gateway和Eureka-client就可以了。截图如下：

​	添加模块：

![image-20200724103944533](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200724103944533.png)

​	选择两个依赖：

![image-20200724104143276](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200724104143276.png)

![image-20200724104212862](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200724104212862.png)

​	后面直接下一步就可以最后把pom文件中的父模块修改为自己的父模块就可以了。剩下就没有什么了。然后就是修改启动类，添加`@EnableDiscoveryClient`注解。最后就是在`application.yml`中增加一些配置。配置如下：

```yaml
server:
  port: 8084
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        - id: eureka-client
          uri: lb://eureka-client
          predicates:
            - Path=/eureka-client/**
    loadbalancer:
      ribbon:
        enabled: false

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8080/eureka
  instance:
    prefer-ip-address: true
```

​	下面说一下这个配置文件需要注意的事。

* uri后面填写的是你服务的地址，例如:`http://www.baidu.com`。这里填写`lb://eureka-client`代表从注册中心获取`eureka-client`的服务。

* predicates是过滤原则，不能为空，必须要填。可以从其他地方了解一个这个如何填。我这里的意思是只要url为：`http://127.0.0.1:8084/eureka-client/**`的请求都会转到对应的服务上去处理。

* id就是id，没有啥特殊的含义或者用处。

* 转到对应的服务时，请求地址会有一定的变化，还是上面那个链接，如果请求的服务端口为8081，则这个请求经过网关路由转化为：`http://127.0.0.1:8081/eureka-client/**`=，其实就是把端口号换了一下，所以我在服务模块添加了一个全局请求前缀：`/rureka-clien`。

* 下面的eureka客户端的配置没有什么可以讲的，和其他的一样。

* 启动的时候报了一个警告，大意是提醒已经启用了Ribbon，可能会有冲突。所以添加了配置：``spring.cloud.loadbalancer.ribbon.enabled`=false`，这里是为了不让这个告警出现（强迫症），要看自己项目是否需要。

  ```java
  You already have RibbonLoadBalancerClient on your classpath. It will be used by default. As Spring Cloud Ribbon is in maintenance mode. We recommend switching to BlockingLoadBalancerClient instead. In order to use it, set the value of `spring.cloud.loadbalancer.ribbon.enabled` to `false` or remove spring-cloud-starter-netflix-ribbon from your project.
  ```

​	剩下就没有什么可以讲的了，启动项目测试一下就可以了，看看最后服务会不会经过路由转换为相应的服务。我最后是测试成功的。

![image-20200724105846220](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200724105846220.png)





​	SpringCloud的内容就暂时告一段落了，这个主要还是熟悉SpringCloud，了解它的一些基础概念，具体到实际开发可能有很大的差异，例如单点登录的整合和跨域的处理，这些我都没有写。所以主要还是作为一个练手项目，了解就可以了。后续还是会持续更新的。（坑多不愁）。

​	就这样吧，结束。

