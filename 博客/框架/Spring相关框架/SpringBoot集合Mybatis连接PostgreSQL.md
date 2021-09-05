## SpringBoot集合MyBatis连接PostgreSQL

​	正好最近需要使用PostgreSQL，蹭着这个机会把MyBatis练习一下。下面就直接开始。	

​	本次也顺带把配置文件修改为yml文件，方便后面的修改和配置。改文件很简单，只需要修改配置文件后缀名为yml即可。Spring可以直接识别，不需要修改其他的配置文件内容。将`application.properties`修改为`application.yml`。同时需要注意两个文件的格式是不一样的。需要注意的就是`:`后面有一个空格。

```properties
#################################### common config : ####################################
server:
  port: 8080
spring:
  application:
    name: train
  #Redis依赖
  # Redis服务器地址
  redis:
    host: 127.0.0.1
    # Redis数据库索引（默认为0）
    database: 0
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    password: mima
    # 连接超时时间（毫秒）
    timeout: 1000
    lettuce:
      pool:
        ## 连接池最大连接数（使用负值表示没有限制）
        max-active: 200
        ## 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1
        ## 连接池中的最大空闲连接
        max-idle: 10
        ## 连接池中的最小空闲连接
        min-idle: 0
  datasource:
    #PostgreSQL配置
    url: jdbc:postgresql://127.0.0.1:5432/postgres
    username: postgres
    password: mima
    driver-class-name: org.postgresql.Driver
    #druid连接池配置
    type: com.alibaba.druid.pool.DruidDataSource
    # 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时
    initialSize: 5
    minIdle: 5
    # 最大连接池数量
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500

mybatis:
	# 加载配置文件
  mapper-locations: classpath:mapper/*.xml
  # 实体类所在路径，填写改路径mapper文件中可以不写全路径
  type-aliases-package: com.psq.train.mysql
```

​	另外，这次也使用了连接池Druid，这个是阿里开发的一个工具，挺好用（大家都这样说）。这次也顺带整合进去。下面就一步一步来吧。

​	首先整合PostgreSQL，添加PostgreSQL依赖即可。如下：

```xml
<!-- postgresql依赖 -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

​	顺带添加Mybatis和Druid的依赖。

```xml
<!-- druid连接池依赖 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.22</version>
</dependency>
<!-- MyBatis依赖 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
```

​	要记得删除之前使用的MySQL的依赖，因为我之前的项目使用了MySQL，所以删除了MySQL和JDBC的依赖。另外这里也遇到一个坑。不知道IDE抽什么风，项目启动的时候一直报错，显示Redis和Mybatis没有配置，但是我依赖都已经删除了。我找了三个小时，最后才发现项目的库不对，仍然存在已经删除的jar包。所以再删除依赖后更新pom文件后，需要查看下面两个地方是不是正常。如下图：

​	![image-20200612161516325](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200612161516325.png)

![image-20200612161543564](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200612161543564.png)

​	更新完毕依赖后，下面就可以编写配置文件了。配置文件如上面所示，PostgreSQL需要注意`url`和`driver-class-name`。从`type`往下都是Druid的配置。Mybatis也是最下面的配置。

​	Druid的配置详细解读这个链接比较详细。[https://www.cnblogs.com/lzhya/p/12493974.html](https://www.cnblogs.com/lzhya/p/12493974.html)

​	同时，因为spring中的Druid配置文件不能读取，所以要写一个配置类，让参数正常加载。代码如下：

```java
package com.psq.train.dao;

import com.psq.train.mysql.TestUser;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * UserMapper.java
 * Description: user表的mapper映射
 *
 * @author Peng Shiquan
 * @date 2020/6/12
 */

public interface UserMapper {

    /**
     * Description: 查询所有用户信息
     *
     * @param
     * @return java.util.List<com.psq.train.mysql.TestUser>
     * @Author: Peng Shiquan
     * @Date: 2020/6/12
     */
    List<TestUser> getAllUser();
}
```

​	Mybatis的配置这里不是很了解，不再解释，后面慢慢补上。

​	为了让dao层的mapper映射文件能够注入，启动类添加了下面的注解。

```java
package com.psq.train;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Description: SpringBoot启动类
 *
 * @param null
 * @return
 * @Author: Peng Shiquan
 * @Date: 2020/5/28
 */
@SpringBootApplication
@MapperScan("com.psq.train.dao")
public class TrainApplication {
    public static void main(String[] args) {
        SpringApplication.run(TrainApplication.class, args);
    }
}
```

​	项目结构图如下：

![image-20200612163659714](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200612163659925.png)

​	下面就是具体的业务代码实现，上面都是基础的配置。

* 首先需要先创建表，这里不再叙述，后面需要补充，我就是简单的创建了一个user表，和MySQL的类似。

* 再就是创建mapper文件，相当于把数据库中的表和java代码联系起来。代码如下：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  
  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.psq.train.dao.UserMapper">
      <select id="getAllUser" resultType="TestUser">
          SELECT id,name,password FROM test_user
      </select>
  </mapper>
  ```

* 创建一个mapper类，方法名要和mapper文件中的id保持一致。代码如下：

  ```java
  package com.psq.train.dao;
  
  import com.psq.train.mysql.TestUser;
  
  import java.util.List;
  
  /**
   * UserMapper.java
   * Description: user表的mapper映射
   *
   * @author Peng Shiquan
   * @date 2020/6/12
   */
  
  public interface UserMapper {
  
      /**
       * Description: 查询所有用户信息
       *
       * @param
       * @return java.util.List<com.psq.train.mysql.TestUser>
       * @Author: Peng Shiquan
       * @Date: 2020/6/12
       */
      List<TestUser> getAllUser();
  }
  ```

* 创建测试类，调用一下mapper类查询一下数据，这里最简单就可以了，所以没有按照web开发的模版写。

  ```java
  package com.psq.train.postgresql;
  
  import com.psq.train.dao.UserMapper;
  import com.psq.train.mysql.TestUser;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Component;
  
  import javax.annotation.PostConstruct;
  import java.util.List;
  
  /**
   * PostgreSQLTest.javaa
   * Description:  PostgreSQL的测试demo
   *
   * @author Peng Shiquan
   * @date 2020/6/12
   */
  @Component
  public class PostgreSQLTest {
  
      @Autowired
      private UserMapper userMapper;
  
      /**
       * Description: PostgreSQL的测试方法
       *
       * @param
       * @return void
       * @Author: Peng Shiquan
       * @Date: 2020/6/12
       */
      @PostConstruct
      public void selectAllUser() {
          List<TestUser> testUserList = userMapper.getAllUser();
          for (TestUser testUser : testUserList) {
              System.err.println(testUser.toString());
          }
      }
  
  }
  ```

​	这里有一个地方需要注意，编写Drudi配置类的时候，发现无法启动，报错信息是注入了相同name的bean，最后排查到是Druid配置类名字和其中一个方法名字相同，导致无法注入。后面查询到这俩注解的作用如下：

* @Configuration

  * @Configuration注解底层是含有@Component ，所以@Configuration 具有和 @Component 的作用。

  * @Configuration注解相当于spring的xml配置文件中<beans>标签，里面可以配置bean。

* @Bean

  * @Bean注解相当于spring的xml配置文件<bean>标签，告诉容器注入一个bean。

  * @Bean注解的方法上如果没通过bean指定实例名，默认实例名与方法名相同。

  * @Bean注解默认为单例模式，可以通过@Scope("prototype")设置为多例。

​	关于PostgreSQL数据库的其他信息，下面一篇博客再说吧，要不然博客就太多了。

​	就这样吧，结束。