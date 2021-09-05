## Dubbo简单Demo搭建

​	上一篇博客简单介绍了一下Dubbo的一些背景，这一篇博客就开始上手一个简单Demo，只有成功的跑起来，才有继续学下去的动力，一直看资料多乏味啊。

### 搭建注册中心

​	Dubbo是使用注册中心来注册服务和发现服务的。上一篇博客里面也有一个Dubbo结构图。挺形象的。Dubbo支持很多种注册中心，Nacos、Zookeeper、Multicast、Redis、Simple。官方推荐Zookeeper注册中心，我们这里也是搭建一个Zookeeper集群。其他的注册中心大家可以去官网了解一下。

​	Zookeeper的搭建因为之前学习过Docker，我这里直接就看了官方的文档，参考写了一个Compose文件，直接启动，下载镜像就可以了，很简单，也不需要很复杂的安装很多东西。这里贴上Compose文件，大家直接运行`docker-compose -f dubbo-compose.yml up -d`即可。

```yaml
version: '3.1'

services:
  zoo1:
    container_name: zoo1
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    networks:
      - dubbo-net
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo2:
    container_name: zoo2
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    networks:
      - dubbo-net
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo3:
    container_name: zoo3
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    networks:
      - dubbo-net
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
  
  dubbo-admin:
    image: apache/dubbo-admin
    container_name: dubbo-admin
    ports:
      - 8080:8080
    networks:
      - dubbo-net
    environment:
      - admin.registry.address=zookeeper://zoo1:2181?backup=zoo2:2181,zoo3:2181
      - admin.config-center=zookeeper://zoo1:2181?backup=zoo2:2181,zoo3:2181
      - admin.metadata-report.address=zookeeper://zoo1:2181?backup=zoo2:2181,zoo3:2181
    depends_on:
      - zoo1
      - zoo2
      - zoo3
networks:
  dubbo-net:
    name: dubbo-net
    driver: bridge
```

​	关于Zookeeper的一些知识后面有机会再学习吧，这一块感觉还挺有意思的。

### 搭建监控中心

​	可以看到上面不止启动了一个容器，还有一个dubbo-admin的镜像，这个镜像就是dubbo的管理平台，但是它的界面是这个样子。

![image-20210824224955559](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210824224955559.png)

​	和我最开始接触老版的管理界面不一样，老版的页面看GitHub上已经不再更新了，界面如下：

![image-20210824225323418](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210824225323418.png)

​	这个监控平台也很好弄，从官网下载老版代码，修改对应的配置，然后直接`java -jar dubbo-admin-0.0.1-SNAPSHOT.jar `即可。地址如下：[老版监控平台代码仓库](https://github.com/apache/dubbo-admin/tree/master-0.2.0)。

​	需要注意，配置文件修改Zookeeper地址时，修改为如下形式。

```yaml
dubbo.registry.address=zookeeper://127.0.0.1:2181?backup=127.0.0.1:2181,127.0.0.1:2181
```

### 项目结构

​	先说一下项目的构成。是一个maven项目，里面有三个子模块，分别是API、生产者和消费者。结构图如下：

![image-20210824230320055](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210824230320055.png)

* API模块：如果生产者需要被多个消费者调用，接口在这些项目中粘贴也不太好，这里将接口定义在API模块中，这样，其他项目直接倒入依赖就可以了，你还可以将这个API打包到你的本地maven仓库。
* consumer模块：就是消费者模块，和前端交互的模块。
* provider模块：生产者模块，业务的主要实现的地方，例如数据库的查询就是在这个模块。

​	父pom文件如下：因为用到的东西比较少，所以依赖不是很多。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.3</version>
        <relativePath/>
    </parent>
    <groupId>com.psq</groupId>
    <artifactId>springboot-exercise</artifactId>
    <version>1.0-SNAPSHOT</version>
    <name>springboot-exercise</name>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <packaging>pom</packaging>

    <modules>
        <module>dubbo-service-provider</module>
        <module>dubbo-service-consumer</module>
        <module>dubbo-service-api</module>
    </modules>

    <dependencies>
        <!--  个人写代码方便，实际开发不推荐      -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!--   SpringBoot依赖  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>


    </dependencies>

</project>
```

### 生产者代码编写

​	生产者先说pom依赖，如果你导入的是阿里的Dubbo依赖，不需要再导入其他依赖了，但是如果你导入的是apache的Dubbo依赖，你需要再根据你使用的注册中心再导入一个依赖，我这里需要再导入一个zookeeper的依赖，pom文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springboot-exercise</artifactId>
        <groupId>com.psq</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbo-service-provider</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <!--  API依赖      -->
        <dependency>
            <groupId>com.psq</groupId>
            <artifactId>dubbo-service-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
            <!-- 导入这个依赖就不需要再导入其他依赖了 -->
<!--        <dependency>-->
<!--            <groupId>com.alibaba.boot</groupId>-->
<!--            <artifactId>dubbo-spring-boot-starter</artifactId>-->
<!--            <version>0.2.0</version>-->
<!--        </dependency>-->
        <!--  下面是导入apacheDubbo的方式     -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <type>pom</type>
            <version>2.7.5</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>

</project>
```

​	生产者还需要再配置一些Dubbo的配置，配置文件代码如下：

```yaml
spring:
  application:
    name: dubbo-service-provider
dubbo:
  application:
    name: dubbo-service-provider
  registry:
    address: zookeeper://127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
    protocol: dubbo
    port: 20880
  scan:
    base-packages: com.psq.dubbboserviceprovider.server
```

​	需要注意你的applicationname不能和消费者一样，最好这个服务唯一，多个相同生产者相同。zookeeper需要注意集群的配置形式，其他的配置也可以看看官方的文档。如下：[Dubbo官方配置文档](https://dubbo.apache.org/zh/docs/references/xml/dubbo-registry/)。

​	这里只是简单demo，就打印一个hello world吧，一个接口，只是返回一个字符串。API的接口代码如下：

```java
package com.psq.dubboserviceapi.service;

/**
 * com.psq.dubboserviceapi.service.SayHelloService.java
 * Description: 示例接口
 *
 * @author Peng Shiquan
 * @date 2021/8/15
 */
public interface SayHelloService {
    /**
     * Description:示例方法
     *
     * @param
     * @return java.lang.String
     * @Author Peng Shiquan
     * @Date 2021-08-15
     */
    String sayHello();
}
```

​	实现方法如下：

```java
package com.psq.dubbboserviceprovider.server;

import com.psq.dubboserviceapi.service.SayHelloService;
import org.apache.dubbo.config.annotation.Service;

/**
 * com.psq.dubbboserviceprovider.server.SayHelloServiceImpl.java
 * Description: SayHelloServiceImpl
 *
 * @author Peng Shiquan
 * @date 2021/8/15
 */
@Service
public class SayHelloServiceImpl implements SayHelloService {

    @Override
    public String sayHello() {
        return "This dubbo test";
    }
}
```

​	需要注意这里的service注解不是spring的注解。是Dubbo的注解。

### 消费者代码编写

​	消费者代码相差不多，pom文件和生产者类似，除了多了一个webstarter，这里就不再贴上代码了。

​	配置文件代码如下：

```yaml
server:
  port: 8081
spring:
  application:
    name: dubbo-service-consumer

dubbo:
  application:
    name: dubbo-service-consumer
  registry:
    address: zookeeper://127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
    protocol: dubbo
    port: 20881
```

​	需要注意不要和你的监控中心端口冲突。其他就没有需要注意的了。

​	这里就简单写一个http请求接口，直接访问生产者的代码，具体代码如下：

```java
package com.psq.dubboserviceconsumer.controlle;

import com.psq.dubboserviceapi.service.SayHelloService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * SayHelloController.java
 * Description:
 *
 * @author Peng Shiquan
 * @date 2021/8/21
 */
@RestController
public class SayHelloController {

    @Reference
    private SayHelloService sayHelloService;

    @GetMapping("/sayHello")
    public String sayHello() {
        String result = sayHelloService.sayHello();
        return result;
    }
}
```

​	启动两个服务后，访问地址：http://localhost:8081/sayHello出现以下画面即代表调用成功，监控中心也会有对应的记录。

![image-20210824232818441](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210824232818441.png)

![image-20210824232906127](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210824232906127.png)

​	至此，Dubbo的简单Demo就完成了，后续会慢慢补充一下配置的相关信息和对于Dubbo底层的学习。

​	就这样吧，结束。
