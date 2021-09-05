## SpringCloud集成Hystrix

​	之前在SpringCLoud了解的博客简单介绍了Hystrix，现在就来详细介绍一下，顺带编写一个demo。

​	为了防止雪崩现象（一个系统中的错误导致系统大面积不可用），所以要让请求在发生错误时仍然可以维持后续的步骤，就是让操作降级，或者后续再处理。总之就是要维持后续的步骤，让系统在发生错误的情况下仍然可用。（这个感觉需要和业务紧密连接，虽然组件可以达到高可用，但是业务上的后续处理还是需要自己编写相关的业务代码的。）

​	Hystrix的原理。

> * 防止任何单个依赖项耗尽所有容器（例如Tomcat）用户线程。
> *  减少负载并快速失败，而不是排队。
>* 在可行的情况下提供备用，以保护用户免受故障的影响。
>* 使用隔离技术（例如如 bulkhead, swimlane, 和 circuit breaker 模式）来限制任何一种依赖关系的影响。
>* 通过近实时指标，监视和警报优化发现时间。
>* 通过在Hystrix的大多数方面中以低延迟传播配置更改来优化恢复时间，并支持动态属性更改，这使您可以通过低延迟反馈回路进行实时操作修改。
>* 防止整个依赖性客户端执行失败，而不仅仅是网络流量失败。

  为了实现这些目标，Hystrix也做了一些处理。

> * 将对外部系统（或“依赖关系”）的所有调用包装在通常在单独线程中执行的`HystrixCommand`或`HystrixObservableCommand`对象中（这是[命令模式](http://en.wikipedia.org/wiki/Command_pattern)的示例）。
> * 超时呼叫花费的时间超过您定义的阈值。有一个默认的，而是由“属性”，使它们比测量的99.5略高的方式对大多数依赖你自定义设置这些超时个百分点每个依存性的性能。
> * 为每个依赖项维护一个小的线程池（或信号灯）；如果已满，发往该依赖项的请求将立即被拒绝，而不是排队。
> * 测量成功，失败（客户端抛出的异常），超时和线程拒绝。
> * 如果某个服务的错误百分比超过阈值，则使断路器跳闸，以在一段时间内手动或自动停止所有对特定服务的请求。
> * 当请求失败，被拒绝，超时或短路时执行回退逻辑。
> * 几乎实时监控指标和配置更改。

  上面都是翻译自Hystrix的GitHub页面，找了一圈，没有找到官网，只找到这个GitHub页面，上面也有一些文档。GitHub地址如下：

[https://github.com/Netflix/hystrix/wiki](https://github.com/Netflix/hystrix/wiki)

​	然后就是一些图片，关于服务的正常调用和异常调用。
<center>正常的服务之间的调用</center>

![image-20200704171401715](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200704171401715.png)

<center>异常的服务之间的调用</center>

![image-20200704172314545](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200704172314545.png)

​	一个服务的节点阻塞，导致阻塞用户的整个请求，最后产生雪崩现象。

<center>Hystrix之间的调用</center>

![image-20200704172521744](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200704172521744.png)

​	还是那句话，有能力的最好直接看官网文档，比较详细而且比较全。

​	介绍完了大致的原理，下面就开始撸代码。

​	因为feign默认集成了Hystrix，所以这里和上一节的Ribbon都不需要添加新的依赖。直接就可以编写。需要修改的地方有三个，一个是在feign接口代码添加一个回调类，还有就是创建一个回调类，写一些业务逻辑代码。最后就是在配置文件中开启Hystrix，因为feign默认不开启。

​	接口代码修改如下：

```java
package com.psq.eurekaclient.feign;

import com.psq.eurekaclient.hystrix.UserHystrix;
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
@FeignClient(name = "provider-mysql", fallback = UserHystrix.class)
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

​	回调类代码如下：

```java
package com.psq.eurekaclient.hystrix;

import com.psq.eurekaclient.feign.UserFeign;

import java.util.List;

/**
 * UserHystrix.java
 * Description: UserFeign的回调类
 *
 * @author Peng Shiquan
 * @date 2020/7/4
 */
public class UserHystrix implements UserFeign {

    /**
     * Description: 查询的回调方法，里面可以放一些异常后处理机制
     *
     * @param ids
     * @return java.lang.String
     * @Author: Peng Shiquan
     * @Date: 2020/7/4
     */
    @Override
    public String selectUserByID(List<String> ids) {
        return "调用异常，但是可以继续往下执行";
    }
}
```

​	配置代码如下：

```properties
#################################### common config : ####################################
spring:
  application:
    name: eureka-client
server:
  port: 8081
#eureka注册中心的配置
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8080/eureka
  instance:
    prefer-ip-address: true
#日志的级别
logging:
  level:
    com.psq.eurekaclient.feign.UserFeign: debug
#feign默认不开启Hystrix，需要配置开启
feign:
  hystrix:
    enabled: true

```

​	关闭生产者前的调用截图如下：

![image-20200704175652527](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200704175652527.png)

关闭生产者后的调用截图如下：

![image-20200704180406092](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200704180406092.png)

​	以上就是简单的调用，说了这么多，这些组件都是为了保证服务的高可用，但是写下来后发现，企业里面要保证高可用只有这些事不够的，需要结合自己公司的情况做对应的修改，有可能需要对这些组件的源码修。还是之前说的，公司的实力不够还是不要搞微服务了，很坑。

​	上面简单的调用没有什么技术点，后面还是需要看看源码或者分析具体的原理。但是现在显然短短一篇博客是不可能的了，这些都可以算到后面的债。

​	就这样吧，结束。

