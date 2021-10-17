## SpringBoot整合Elasticsearch

​	又是项目优化，之前的ES整合的版本因为太老了，所以想换成新的，顺带看看有没有什么新的写法。直接开整吧。

​	最开始其实需要先介绍一下Spring Data，Spring Data的任务是为数据访问提供一个熟悉且一致的，基于Spring的编程模型，同时仍保留基础数据存储的特殊特征。它使使用数据访问技术，关系和非关系数据库，map-reduce框架以及基于云的数据服务变得容易。这是一个总括项目，其中包含许多特定于给定数据库的子项目。这些项目是与这些令人兴奋的技术背后的许多公司和开发人员共同开发的。

​	上面的介绍来自于官网，官网地址如下：[https://spring.io/projects/spring-data](https://spring.io/projects/spring-data)

​	其实说白了，Spring Data就是Spring官方将各个中间件整合，对中间件的操作进行了二次封装，让使用者可以更快的使用这些中间件。这样做有好有坏，好处是使用者不用关心底层的实现逻辑，甚至不用关心中间件的特点，可以快速上手。坏处也是这点，因为封装太多，一旦底层暴露出一些BUG，使用者完全不知道如何处理。这不仅仅是Spring Data的痛点，感觉是Spring全家桶的通病。感觉有点扯远了，开始整合Elasticsearch。

​	Spring Data中的Elasticsearch模块有着一下特点：

- Spring配置支持使用基于Java的`@Configuration`类或ES客户端实例的XML名称空间。
- `ElasticsearchTemplate`帮助程序类，可提高执行常规ES操作的效率。包括文档和POJO之间的集成对象映射。
- 与Spring的转换服务集成的功能丰富的对象映射。
- 基于注释的映射元数据，但可扩展以支持其他元数据格式。
- `Repository`接口的自动实现，包括对自定义查找器方法的支持。
- CDI对存储库的支持。

​	可以看到还是有很多比较好的点，也是方便上手和开发。下面就开始编写代码，这里使用的都是最新的版本（注意）。

​	同时，这些代码都是看了一些其他博客和官网的例子。

​	官网文档地址：[https://docs.spring.io/spring-data/elasticsearch/docs/4.0.1.RELEASE/reference/html/#elasticsearch.clients](https://docs.spring.io/spring-data/elasticsearch/docs/4.0.1.RELEASE/reference/html/#elasticsearch.clients)

​	需要先将项目导入Spring Data Elasticsearch的依赖，这里SpringBoot的版本是：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

​	导入的依赖让SpringBoot自动选择，依赖如下：

```xml
<!--  ES依赖      -->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
</dependency>
```

​	需要添加一个配置类，用来配置ES的基础信息。代码如下：

```java
package com.psq.train.config;

import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.elasticsearch.client.ClientConfiguration;
import org.springframework.data.elasticsearch.client.RestClients;
import org.springframework.data.elasticsearch.config.AbstractElasticsearchConfiguration;
import org.springframework.data.elasticsearch.repository.config.EnableElasticsearchRepositories;

/**
 * RestClientConfig.java
 * Description:  ESclient的配置类
 *
 * @author Peng Shiquan
 * @date 2020/7/20
 */
@Configuration
@EnableElasticsearchRepositories(basePackages = "com.psq.train.repository")
public class RestClientConfig extends AbstractElasticsearchConfiguration {

    @Bean
    @Override
    public RestHighLevelClient elasticsearchClient() {
        /**
         * 使用构造器来提供集群地址，设置默认值或者启用SSL
         */
        final ClientConfiguration clientConfiguration = ClientConfiguration.builder()
                .connectedTo("111.229.157.173:9200")
                .build();
        /**
         * 创建RestHighLevelClient
         */
        return RestClients.create(clientConfiguration).rest();
    }
}
```

​	剩下就是需要创建一个实体类和一个存储库。代码分别如下：

​	实体类代码如下：

```java
package com.psq.train.repository;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;

/**
 * TestPerson.java
 * Description:  ES测试实体类
 *
 * @author Peng Shiquan
 * @date 2020/7/20
 */
@Document(indexName = "test_person")
public class TestPerson {

    @Id
    private String id;
    private String firstName;
    private String lastName;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return "TestPerson{" +
                "id='" + id + '\'' +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                '}';
    }
}
```

​	存储库代码如下：

```java
package com.psq.train.repository;

import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

/**
 * TestPersonRepository.java
 * Description:  TestPerson实体类的Repository
 *
 * @author Peng Shiquan
 * @date 2020/7/20
 */
public interface TestPersonRepository extends ElasticsearchRepository<TestPerson, String> {
}
```

Controller层调用代码：

```java
@RequestMapping(value = "/elasticsearch", method = RequestMethod.GET)
public String testES() {
    TestPerson testPerson = new TestPerson();
    testPerson.setFirstName("psq");
    testPerson.setLastName("haha");
    TestPerson saveResult = testPersonRepository.save(testPerson);
    PageRequest pageRequest = PageRequest.of(0, 5);
    Page<TestPerson> testPersonPage = testPersonRepository.findAll(pageRequest);
    return testPersonPage.toString();
}
```

代码都上完了，下面就说说要注意的点。

* 因为这里只是简单使用，所以最开始的ES配置类添加的东西很少，只是创建了一个`RestHighLevelClient`，剩下的就不是很了解了，所以也没有去编写代码。有大佬知道的也可以告知，非常欢迎补足。

  ![image-20200722102642051](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200722102642051.png)

* 代码的这个地方使用的9200端口，并且没有设置集群名称。是因为9200端口是作为Http协议，主要用于外部通讯。9300端口作为Tcp协议，jar之间就是bai通过tcp协议通讯。所以使用9300是需要指定集群名称的，使用9200则不需要指定名称的。

* `RestHighLevelClient`是直接使用http协议来操作ES的，所以官方会推荐这个客户端，不再推荐使用`TransportClient`。

* 本次代码使用的ES服务库是7.8版本的，因为低版本的ES会报一些错误，不识别部分参数。如果遇到低版本的ES，切换依赖版本就可以了。

* Spring Data Elasticsearch其实还有一个优点，不过代码中没有体现出来。官网文档说的挺明白了，就是根据方法名自动生成执行方法，不需要再写实现类。同时，这里估计也可以添加分页的参数。不过官网对于分页说的比较少，需要大家到博客上找一找。

  ![image-20200722103750821](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200722103750821.png)

* 这里直接指定了index，但是type没有指定，后续Elasticsearch会去掉这个属性。项目启动的时候，如果没有这个index，插入的时候会自动创建。

  
	

​	剩下就没有什么可以说的，还是那句话，后续再写的时候再去学习和深入。

​	就这样吧，结束。