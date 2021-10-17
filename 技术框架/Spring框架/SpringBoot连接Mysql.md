## SpringBoot连接Mysql

​	第一个SpringBoot的练习就是连接数据库，使用的是就是基础的组件，没有使用web和mybatis。也是作为一个基础的demo。开始。

​	因为之前的java项目是通过导入jar包来实现，非常的繁琐并且jar包不容易管理。后来产生了maven，通过maven来管理jar包。本次项目也是通过pom文件来管理依赖，同时，因为SpringBoot设置了默认的依赖版本，这里也不再叙述导入依赖的版本。

![image-20200530152138145](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200530152138145.png)

​	导入的依赖如下：下面的代码就是全部的依赖，有些最开始创建项目的时候已经导入了，这里仍然放在下面了。记得添加依赖后要更新一下，要不然很多东西IDE无法做配置。

```xml
<!-- SpringBoot基础依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<!-- JDBC依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<!-- MySQL依赖 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>

<!-- 测试依赖 -->
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
```

​	maven更新的图片如下，这样更新后就可以使用依赖中的类。

![image-20200531130454654](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200531130454654.png)	

​	导入依赖后，需要在SpringBoot配置文件中进行数据库的基础配置。这里配置文件是application.properties。不是现在常见的yml格式，这里是spring默认的格式。其他格式的数据库配置需要查询一下，后面也会换成其他格式，这种格式不好区分各个中间件配置，但是现在暂时使用这个格式。配置如下：

```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test
spring.datasource.username=root
spring.datasource.password=12345678
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

​	创建测试类就可以了，下面就可以写代码了。代码如下：

```java
package com.psq.train.mysql;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

/**
 * ConnectMySQLTrain.java
 * Description: 使用基础的组件连接MySQL
 *
 * @author Peng Shiquan
 * @date 2020/5/30345
 */
@Component
public class ConnectMySQLTrain {


    @Resource
    private JdbcTemplate jdbcTemplate;

    /**
     * Description: 测试MySQL数据读取
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/5/31
     */
    @PostConstruct
    public void testMySQL() {
        String sql = "SELECT id,name,password FROM `user`";
        List<TestUser> testUserList = jdbcTemplate.query(sql, new RowMapper<TestUser>() {
            @Override
            public TestUser mapRow(ResultSet resultSet, int i) throws SQLException {
                TestUser testUser = new TestUser();
                testUser.setId(resultSet.getInt("id"));
                testUser.setName(resultSet.getString("name"));
                testUser.setPassword(resultSet.getString("password"));
                return testUser;
            }
        });
        System.err.println("查询成功，查询结果如下：" + testUserList.toString());
    }

}
```

​	需要注意的是：这里没有使用SPringBoot的web模块，只是简单的一个demo，所以就单独写了一个类，在项目启动的时候运行一下就可以了`@Component`注解是为了把普通pojo实例化到spring容器中，相当于配置文件中的`<bean id="" class=""/>`，`@PostConstruct`注解的意思是：被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。实体类的代码因为没有什么可以讲的，这里就不再占面积了。

​	运行项目这里也没有什么可以讲的地方。就是SpringBoot内置了Tomcat，但是因为我这个项目没有web模块，所以没有内嵌tomcat，关于这些可以放到后面来说。本次就是打成了jar包，通过jvm运行。结果如下：

![image-20200531132745688](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200531132745688.png)

​	上面说的可能有些地方不对，大家可以指导一下。就这样吧，结束。