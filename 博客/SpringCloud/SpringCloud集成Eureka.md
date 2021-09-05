## SpringCloud集成Eureka

​	下面就开始简单的写一个demo，先把组件慢慢都集成进来，先集成Eureka。

### 创建父模块

​	父模块需要创建maven项目，这里没有找到基础的方法，就是不使用任何IDE去生成项目，所以最后无奈只能使用IDEA来生成这些项目。

​	首先选择maven，再输入自己项目的信息。

![image-20200602152019621](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200602152019621.png)


![image-20200602141047248](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200602141047248.png)

​	输入完成后，删除src目录，只保留一个pom文件，加入SpringBoot依赖即可。

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

### 创建Eureka服务注册中心

​	新建一个模块，根据spring的配置添加相应的依赖即可。这里需要注意一下。

​	我的IDE不知道抽什么风，一直连接不上springio，IDEA的代理配置能测试成功，但是到创建模块时死活不行，修改代理、用VPN、改地址为http，网上能找到的方法都试了，还是不行。最后放弃了，用的阿里云的spring镜像服务器。如下：

![image-20200602142009357](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200602142009357.png)

​		中间有些步骤我给省略了，没有什么可讲的地方，下面就是选择依赖。

![image-20200602142110037](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200602142110037.png)

​	后面就创建成功了，修改Eureka的配置即可。修改如下。

#### pom文件

​	pom文件需要修改父类依赖指向，指向刚才创建的父模块。EurekaServer依赖刚才创建模块的时候就已经导入了，不需要了。另外，看自己的需求导入健康检查依赖，这里我没有使用，就没有导入。pom文件全部代码如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.psq</groupId>
        <artifactId>ssodemo</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <groupId>com.psq</groupId>
    <artifactId>eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka</name>
    <description>this is Eureka Server</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR4</spring-cloud.version>
    </properties>

    <dependencies>
        <!-- eureka依赖       -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 健康检查依赖       -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

#### application.yml文件

​	application.yml文件需要对项目做一个基础的配置，这里按照自己的需求配置就可以了。因为只是引入了Eureka，所以配置文件很少。另外，刚创建项目的时候是application.properties，命名为application.yml文件即可，要注意这两个文件虽然Spring都支持，但是这俩种文件的书写格式不一样。配置文件代码如下：

```properties
#################################### common config : ####################################
spring:
  application:
    name: eureka

server:
  port: 8080

##eureka服务段配置
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://127.0.0.1:8080/eureka
  server:
    renewal-percent-threshold: 0.49
```

#### 启动类添加Eureka注解

​	启动类添加注解即可，添加一个`@EnableEurekaServer`注解，然后启动服务就可以了。

#### 启动结果及其验证方法

​	启动单个模块，然后在浏览器输入[http://127.0.0.1:8080](http://127.0.0.1:8080)，出现下面的页面就代表成功了。

![截屏2020-06-02 下午2.59.01](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/截屏2020-06-02%20下午2.59.01.png)

### 创建Eureka客户端

​	创建模块的步骤和上面一致，不过依赖和相应的配置需要修改一下。

#### pom文件

​	没有什么可以讲的地方。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.psq</groupId>
        <artifactId>ssodemo</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <groupId>com.psq</groupId>
    <artifactId>eureka-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>eureka-client</name>
    <description>this is Eireka Client</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR4</spring-cloud.version>
    </properties>

    <dependencies>
        <!--web依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

#### application.yml配置文件

​	唯一需要注意的地方就是端口号和注册中心的地址。

```properties
#################################### common config : ####################################
spring:
  application:
    name: eureka-client
server:
  port: 8081
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:8080/eureka
  instance:
    prefer-ip-address: true
```

#### 启动类注解

​	和注册中心的注解也不一样，需要修改一下。使用`@EnableDiscoveryClient`这个注解。

#### 启动监测

​	需要注意的是，如果依赖中没有添加web依赖，项目就会启动一会就停掉了。因为没有添加web依赖的SpringBoot项目相当于一个java的demo，代码执行完毕就结束了。执行项目后，注册中心回出现已经注册的实例。如下图：

![image-20200602151510377](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200602151510377.png)

​	这样就整合了一个Eureka，一个简单的demo，中间省略了很多的东西。只是简单的验证一下。就这样吧，结束。