## SpringBoot连接Redis

​	下面就开始练习连接Redis，这里也是没有集成web模块。web模块感觉这些东西也没有必要。连接池这里可以讲一下，上一篇MySQL的连接池放在Mybatis那里再讲，估计也快了，如果不出意外的话。。。

​	首先安装Redis和配置Redis的信息，这里不再累述，我之前的博客也有。刚才看了一下，我之前的博客讲的还挺详细的，我这次有可能重复了，尽量讲讲之前没有讲到的。

​	仍然是添加依赖，因为Redis官方开发了一个连接工具Jedis，这个工具挺好用的。

```xml
        <!--  Redis依赖      -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--  不添加这个依赖会导致RedisTemplate无法注入      -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
```

​	下面就是配置Redis连接和连接池信息。

```properties
#Redis依赖
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=自己的密码
# 连接超时时间（毫秒）
spring.redis.timeout=1000
## 连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=200
## 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=-1
## 连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=10
## 连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
```

​	下面就开始编写代码，需要注意的是代码中引用的类是StringRedisTemplate，有一些不同的地方。

* RedisTemplate，RedisTemplate是最基本的操作类，它默认的序列化方式是JdkSerializationRedisSerializer，在存值时，键值会被序列化为字节数组，可读性差，取值时也是一样，如果redis中存的值正常的字符串形式，取值时将返回null。

* StringRedisTemplateStringRedisTemplate继承于 RedisTemplate<String, String>，默认的序列化方式是StringRedisSerializer，存值取值都是按照字符串的形式。

  代码如下，但是中间遇到了一些问题，解决起来很蛋疼。下面说说，最开始使用的是上面的依赖，但是启动一直报错，一开始怀疑注入的方式不对，把`@Resource`换成`@Autowired`，结果还是不行，又怀疑方法不能正常调用（菜鸟的智障怀疑），后来又排除了。在网上找了一圈，怀疑redis版本的问题，换成了jedis，结果就可以了。然后就认为是依赖的版本问题，生菜（最新redis使用的）和目前的SpringBoot融合的不是很好。

  ```java
  package com.psq.train.redis;
  
  import com.psq.train.mysql.TestUser;
  import org.springframework.data.redis.core.StringRedisTemplate;
  import org.springframework.stereotype.Component;
  
  import javax.annotation.PostConstruct;
  import javax.annotation.Resource;
  
  /**
   * RedisTrain.java
   * Description: Redis练习
   *
   * @author Peng Shiquan
   * @date 2020/5/31
   */
  @Component
  public class RedisTrain {
  
      /**
       * 使用StringRedisTemplate，类似于RedisTemplate<String,String>
       */
      @Resource
      private StringRedisTemplate stringRedisTemplate;
  
      /**
       * Description: Redis练习的主方法
       *
       * @param
       * @return void
       * @Author: Peng Shiquan
       * @Date: 2020/5/31
       */
      @PostConstruct
      public void redisTrainmain() {
          TestUser testUser = new TestUser();
          testUser.setId(3);
          testUser.setName("张三");
          testUser.setPassword("1234567");
          System.err.println("下面开始Redis插入");
          redisTrainInsert(testUser);
          System.err.println("下面开始查询");
          redisTrainSelect(testUser.getName());
      }
  
      /**
       * Description: redis插入方法
       *
       * @param testUser
       * @return void
       * @Author: Peng Shiquan
       * @Date: 2020/6/1
       */
      public void redisTrainInsert(TestUser testUser) {
          String key = testUser.getName();
          String value = testUser.toString();
          stringRedisTemplate.opsForValue().set(key, value);
          System.err.println("存储成功，存储对象为:key:" + key + ",value:" + value);
      }
  
      /**
       * Description: redis查询方法
       *
       * @param name
       * @return void
       * @Author: Peng Shiquan
       * @Date: 2020/6/1
       */
      public void redisTrainSelect(String name) {
          System.err.println("需要查询的name为:" + name);
          System.err.println(name);
          String value = stringRedisTemplate.opsForValue().get(name);
          System.err.println("查询到到结果为:" + value);
      }
  }
  ```

* `Lettuce`和`Jedis`的都是连接`Redis Server`的客户端程序。`Jedis`在**实现上是直连`redis server`，多线程环境下非线程安全，除非使用连接池，为每个Jedis实例增加物理连接**。`Lettuce`基于Netty的连接实例（StatefulRedisConnection），**可以在多个线程间并发访问，且线程安全，满足多线程环境下的并发访问，同时它是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例**。

  我也这样认为，直到我再次启动的时候，发现下面一个问题。  ![image-20200601090901717](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200601090901717.png)

  如上图，最后显示的是一个密码的问题，是客户端配置了密码，但是服务端没有配置密码，我又查看了redis启动的时候，没有使用正确的配置文件。

![image-20200601092939704](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200601092939704.png)

​	配置了密码后，一切都解决了。真是一个SB问题，果然是老了。。。

​	剩下就遗留了一个问题，我看别人的demo都写了一个redis配置类，估计是为了定制自己的`RedisTemplate`，我这里没有什么定制的，就是简单的用了一下，所以没有写redis配置类。如果我说错了，大家也指点一下。

​	就这样吧，结束。

