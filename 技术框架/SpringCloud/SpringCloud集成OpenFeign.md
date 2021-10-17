## SpringCloud集成OpenFeign

​	上次讲完Eureka，这次来讲一下OpenFeign。

​	之前的Eureka没有讲它的原理，后面看看能不能补上，这里也简单叙述一下OpenFeign的原理，后面慢慢补充。

​	OpenFeign。简化调用服务时的工作，将复杂的请求简化为配置，使用动态代理来构造出需要请求的服务地址，最后发起请求和解析请求。

​	微服务即然已经搭建起来了，但是各个微服务之前如何通讯和调用确实各问题，当然可以通过http的形式调用相应的服务，但是这样写的话，比较繁琐，并且代码的耦合度也会很高，这里就引出了Openfeign的概念。Feign通过处理注解，将请求模板化，当实际调用的时候，传入参数，根据参数再应用到请求上，进而转化成真正的请求，这种请求相对而言比较直观。所以一个插件或者组件的产生肯定是为了解决某个问题的，不可能没有任何用处但是仍然在使用。

​	Openfeign的原理相对于我来说还是比较复杂的，这里我也就我自己的理解和别人的博客简单说一下Openfeign的一次调用。

 * 在使用feign 时，会定义对应的接口类，在接口类上使用Http相关的注解，标识HTTP请求参数信息

 * 在Feign 底层，通过基于面向接口的动态代理方式生成实现类，将请求调用委托到动态代理实现类

 * 最后将方法的调用转换为一次http请求

   
   

​	我的理解就是这么个过程，可能不太对，欢迎大家指正。下面就上代码。

### 生产者

​	生产者的代码主要就是一个controller，只要有返回就可以了。这里就不再累述。没有什么可讲的。主要讲一下中间遇到的坑。

* IDEA运行的时候，因为我的结构是父模块和子模块的形式，所以在编译的时候一直没有通过，只需要先把父模块编译，要注意pom文件不要带子模块，编译完毕在`install`一下就可以了，子模块就可以编译运行了。

* 为了方便后面的测试，所以在模块中添加了swagger-ui，很简单，添加pom文件、增加一个swagger配置类就可以了。详细的解释和需要添加的注解可以看下面的俩博客。

   [https://www.cnblogs.com/xifengxiaoma/p/11022146.html](https://www.cnblogs.com/xifengxiaoma/p/11022146.html)

  [https://blog.csdn.net/u012702547/article/details/88775298](https://blog.csdn.net/u012702547/article/details/88775298)

  生产者代码如下：

  ```java
  package com.psq.providermysql.controller;
  
  import io.swagger.annotations.Api;
  import io.swagger.annotations.ApiOperation;
  import org.springframework.web.bind.annotation.*;
  
  import java.util.List;
  
  /**
   * UserController.java
   * Description:  User相关controller
   *
   * @author Peng Shiquan
   * @date 2020/6/16
   */
  @RequestMapping(value = "/user")
  @RestController
  @Api(value = "用户", tags = "用户管理相关接口")
  public class UserController {
  
      @GetMapping(value = "/")
      @ApiOperation(value = "查询用户接口")
      public String selectUserByID(@RequestParam List<String> ids) {
          return "hello SpringCloud";
      }
  
  
  }
  ```

### 消费者

​	主要改动的地方就在消费者，首先消费者模块需要添加一个feign依赖，然后消费者通过编写一个feign接口来代表生产者的方法，controller通过`@Autowired`注解直接使用就可以了，启动类也需要添加一个`@EnableFeignClients`注解。其他的地方就不需要改动了。代码如下：

* Feigin接口代码。

  ```java
  package com.psq.eurekaclient.feign;
  
  import org.springframework.cloud.openfeign.FeignClient;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RequestParam;
  
  import java.util.List;
  
  /**
   * UserFeign.java
   * Description: User的feign
   *
   * @author Peng Shiquan
   * @date 2020/6/16
   */
  @FeignClient(name = "provider-mysql")
  public interface UserFeign {
  
      /**
       * Description: 查询list中的id所对应的用户信息
       *
       * @param ids
       * @return java.lang.String
       * @Author: Peng Shiquan
       * @Date: 2020/6/16
       */
      @GetMapping("/user/")
      String selectUserByID(@RequestParam List<String> ids);
  
  }
  ```

* controller层代码。

  ```java
  package com.psq.eurekaclient.controller;
  
  import com.psq.eurekaclient.feign.UserFeign;
  import io.swagger.annotations.Api;
  import io.swagger.annotations.ApiOperation;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestParam;
  import org.springframework.web.bind.annotation.RestController;
  
  import java.util.List;
  
  /**
   * UserController.java
   * Description:  User相关controller
   *
   * @author Peng Shiquan
   * @date 2020/6/16
   */
  @RequestMapping(value = "/user")
  @RestController
  @Api(value = "用户", tags = "用户管理相关接口")
  public class UserController {
  
      @Autowired
      private UserFeign userFeign;
  
  
      @GetMapping(value = "/")
      @ApiOperation(value = "查询用户接口")
      public String selectUserByID(@RequestParam List<String> ids) {
          String result = userFeign.selectUserByID(ids);
          return result;
      }
  
  
  }
  ```

* 启动类代码。

  ```java
  package com.psq.eurekaclient;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
  import org.springframework.cloud.openfeign.EnableFeignClients;
  
  /**
   * EurekaClientApplication.java
   * Description:  Eureka Client 启动类
   *
   * @author Peng Shiquan
   * @date 2020/6/16
   */
  @EnableDiscoveryClient
  @SpringBootApplication
  @EnableFeignClients
  public class EurekaClientApplication {
      public static void main(String[] args) {
          SpringApplication.run(EurekaClientApplication.class, args);
      }
  
  }
  ```

* 导入的依赖。

  ```xml
  <!--配置feign-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  ```

​	代码就是上面的代码，运行截图如下：

​	![image-20200616145706715](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200616145706715.png)

​	难点上面都说了，也没有啥可以说的了，感觉坑还是自己踩踩比较好，后面也会又个深入的了解，也会把踩到的坑说说。

​	顺带说一下，代码也上传到GitHub了，大家也可以下载源码。链接：[https://github.com/qq1162210866/springcloud-train](https://github.com/qq1162210866/springcloud-train)

​	就这样吧，结束。