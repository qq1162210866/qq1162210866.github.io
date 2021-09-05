## Spring内部知识点

### IOC相关知识点

#### IOC基础概念

​	所谓 IOC ，就是由 Spring IOC 容器来负责对象的生命周期和对象之间的关系。之前传统的是由对象内部直接生成，这样耦合度太高。通过控制反转，只需要告知IOC容器你需要什么样的实例，IOC容器就会自动将实例注入到你的实例里面。控制反转需要依赖注入的支持，即依赖注入是控制反转的实现。依赖注入就是由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。

​	[详细的解释](https://juejin.cn/post/6857406008877121550)

#### 依赖注入的类型和方式

​	依赖注入的类型有：

* 值类型注入 ：就是基础的数据类型，如字符串和Integer

```xml
<!-- applicationContext.xml -->
    <!-- 普通数据类型注入，通过构造器 -->
    <bean id="user1" class="po.User">
        <constructor-arg index="0" value="zhangsan"></constructor-arg>
        <constructor-arg index="1" value="18"></constructor-arg>
    </bean>
    <!-- 普通数据类型注入，通过setter方法 -->
    <bean id="user2" class="po.User">
        <property name="name" value="lisi"></property>
        <property name="age" value="20"></property>
    </bean>
```

* 引用类型注入：需要依赖的是一个实例，需要现在xml文件中创建这个实例的bean，再去引用

```xml
<!-- applicationContext.xml -->
    <!-- 引用数据类型注入，通过构造器 -->
    <bean id="userDao" class="dao.impl.UserDaoImpl"></bean>
    <bean id="userService" class="service.impl.UserServiceImpl">
        <constructor-arg index="0" ref="userDao"></constructor-arg>
    </bean>
    <!-- 引用数据类型注入，通过setter方法 -->
    <bean id="userDao" class="dao.impl.UserDaoImpl"></bean>
    <bean id="userService" class="service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"></property>
    </bean>
```

* 复杂的集合形式：将多个实例放入到集合中，本质和引用类型相差不多，但是需要注意标签。（集合类型的注入只能通过XML来实现）

```xml
<!-- applicationContext.xml -->
    <!-- 集合数据类型注入 -->
    <bean id="user1" class="po.User">
        <property name="name" value="zhangsan"></property>
        <property name="age" value="18"></property>
    </bean>
    <bean id="user2" class="po.User">
        <property name="name" value="lisi"></property>
        <property name="age" value="20"></property>
    </bean>
    <bean id="userDao" class="dao.impl.UserDaoImpl">
        <property name="list">
            <list>
                <value>aaa</value>
                <value>bbb</value>
                <value>ccc</value>
            </list>
            <!-- 另一种方法，需要添加第三方约束 -->
            <!-- xmlns:util="http://www.springframework.org/schema/util" -->
            <!-- http://www.springframework.org/schema/util -->
            <!-- http://www.springframework.org/schema/util/spring-util.xsd -->
            <!--<util:list list-class="java.util.ArrayList">
                <value>aaa</value>
                <value>bbb</value>
                <value>ccc</value>
            </util:list>-->
        </property>
        <property name="set">
            <list>
                <value>ddd</value>
                <value>eee</value>
                <value>fff</value>
            </list>
        </property>
        <property name="userList">
            <list>
                <ref bean="user1"></ref>
                <ref bean="user2"></ref>
            </list>
        </property>
        <property name="userMap">
            <map>
                <entry key="k_user1" value-ref="user1"></entry>
                <entry key="k_user2" value-ref="user2"></entry>
            </map>
        </property>
        <property name="properties">
            <props>
                <prop key="properties_name">wangwu</prop>
                <prop key="properties_age">25</prop>
            </props>
        </property>
    </bean>
```

​	依赖注入的方式有：

* 第一种：使用构造函数提供。
* 第二种：使用set方法提供(常用)。
* 第三种：使用注解提供。首先按照类型去Spring容器中圈定出来匹配的对象，接下来使用变量名作为bean的id在圈定出来的内容里继续查找，如果有一样的也可以注入成功，如果都不一样报错。

​	需要注意的点：

* 一个bean代表一个实例，一个实例只能调用一个构造方法。
* 一个bean里面可以进行组合，可以使用构造方法注入，也可以使用set方法注入。
* 当bean里面有多个注入时，构造方法在前，set在后，会造成覆盖。构造器方法注入->字段注入->属性或普通方法注入.
* 如何想要使用set方法注入，必须实现默认无参构造方法。如果实现了其他的构造方法，则无参构造方法不会自动生成。（Spring需要先利用无参的构造方法反射创建一个对象，再使用set方法给属性赋值）

#### 依赖注入常用的注解

* @Qualifier：在按照类中注入的基础之上再按照名称注入。它在给类成员注入时不能单独使用。但是在给方法参数注入时可以。需要搭配@Autowired
* @Autowired默认按类型装配。
* @Resource（常用）：直接按照bean的id注入。它可以独立使用。
* @Value：用于注入基本类型和String类型的数据。

#### IOC容器

​	IOC 容器具有依赖注入功能的容器，它可以创建对象，IOC 容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。

​	常用的两个容器：

* BeanFactory 容器：最简单的容器，只提供基础的依赖注入的功能。
* ApplicationContext 容器：Spring上下文，比较复杂的容器。提供了其他的功能。

##### Bean的作用域

|     作用域     |                             描述                             |
| :------------: | :----------------------------------------------------------: |
|   singleton    | 在spring IoC容器仅存在一个Bean实例，Bean以单例方式存在，默认值。（不是线程安全的） |
|   prototype    | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean() |
|    request     | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
|    session     | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean，仅适用于WebApplicationContext环境 |
| global-session | 一般用于Portlet应用环境，该作用域仅适用于WebApplicationContext环境 |

##### Bean的生命周期

全套初始化方法及其标准顺序是：

* BeanNameAware 的setBeanName：在创建此 bean 的 bean 工厂中设置 bean 的名称。
* BeanClassLoaderAware 的setBeanClassLoader：将 bean class loader提供给 bean 实例的回调。
* BeanFactoryAware 的setBeanFactory：将拥有工厂提供给 bean 实例的回调。
* EnvironmentAware 的setEnvironment：设置该组件运行的Environment 。
* EmbeddedValueResolverAware 的setEmbeddedValueResolver：设置 StringValueResolver 以用于解析嵌入的定义值。
  ResourceLoaderAware 的setResourceLoader （仅在应用程序上下文中运行时适用）：设置运行此对象的 ResourceLoader。
* ApplicationEventPublisherAware 的setApplicationEventPublisher （仅在应用程序上下文中运行时适用）：设置运行此对象的 ApplicationEventPublisher。
* MessageSourceAware 的setMessageSource （仅在应用程序上下文中运行时适用）：设置此对象运行所在的 MessageSource。
* ApplicationContextAware 的setApplicationContext （仅在应用程序上下文中运行时适用）：设置此对象运行所在的 ApplicationContext。通常此调用将用于初始化对象。
* ServletContextAware 的setServletContext （仅在 web 应用上下文中运行时适用）：设置该对象运行所在的ServletContext 。
* BeanPostProcessors 的postProcessBeforeInitialization方法：在任何 bean 初始化回调（如 InitializingBean 的afterPropertiesSet或自定义初始化方法）之前，将此BeanPostProcessor应用于给定的新 bean 实例。bean 已经被填充了属性值。 返回的 bean 实例可能是原始实例的包装器。默认实现按原样返回给定的bean 。
* InitializingBean 的afterPropertiesSet：在设置所有 bean 属性并满足BeanFactoryAware ， ApplicationContextAware等之后，由包含BeanFactory调用。
* 自定义初始化方法定义：
* BeanPostProcessors 的postProcessAfterInitialization方法：在任何 bean 初始化回调（如 InitializingBean 的afterPropertiesSet或自定义初始化方法）之后，将此BeanPostProcessor应用于给定的新 bean 实例。bean 已经被填充了属性值。 返回的 bean 实例可能是原始实例的包装器。

​	在关闭 bean 工厂时，以下生命周期方法适用：

* DestructionAwareBeanPostProcessors 的postProcessBeforeDestruction方法：在销毁之前将此 BeanPostProcessor 应用于给定的 bean 实例，例如调用自定义销毁回调。
* DisposableBean 的destroy：在销毁 bean 时由包含BeanFactory调用。
* 自定义销毁方法定义：

![image-20210618220738157](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210618220738157.png)

实例化 -> 属性赋值 -> 初始化 -> 销毁

##### 自动装配

​	上面的方式都是通过xml配置bean，非常的复杂和繁琐。自动装配就是解决这个问题，但是自动装配不能装配所谓的简单类型包括基本类型，字符串和类。

​	自动装配的模式：

|            模式             |                             描述                             |
| :-------------------------: | :----------------------------------------------------------: |
|             no              | 这是默认的设置，它意味着没有自动装配，你应该使用显式的bean引用来连线。你不用为了连线做特殊的事。在依赖注入章节你已经看到这个了。 |
|           byName            | 由属性名自动装配。Spring 容器看到在 XML 配置文件中 bean 的自动装配的属性设置为 byName。然后尝试匹配，并且将它的属性与在配置文件中被定义为相同名称的 beans 的属性进行连接。 |
|           byType            | 由属性数据类型自动装配。Spring 容器看到在 XML 配置文件中 bean 的自动装配的属性设置为 byType。然后如果它的**类型**匹配配置文件中的一个确切的 bean 名称，它将尝试匹配和连接属性的类型。如果存在不止一个这样的 bean，则一个致命的异常将会被抛出。 |
|         constructor         | 类似于 byType，但该类型适用于构造函数参数类型。如果在容器中没有一个构造函数参数类型的 bean，则一个致命错误将会发生。 |
| autodetect（3.0版本不支持） | Spring首先尝试通过 constructor 使用自动装配来连接，如果它不执行，Spring 尝试通过 byType 来自动装配。 |

### AOP相关知识点

​	面向方面的编程需要把程序逻辑分解成不同的部分称为所谓的关注点。跨一个应用程序的多个点的功能被称为横切关注点，这些横切关注点在概念上独立于应用程序的业务逻辑。有各种各样的常见的很好的方面的例子，如日志记录、审计、声明式事务、安全性和缓存等。

​	当出现多个对象需要一个公共行为时，在每个对象里面编写也是不太现实的，所以引入了AOP。AOP有两种实现方式：基于JDK和CGLIB。

​	AOP中的几个概念：

|      项       |                             描述                             |
| :-----------: | :----------------------------------------------------------: |
|    Aspect     | 一个模块具有一组提供横切需求的 APIs。例如，一个日志模块为了记录日志将被 AOP 方面调用。应用程序可以拥有任意数量的方面，这取决于需求。 |
|  Join point   | 在你的应用程序中它代表一个点，你可以在插件 AOP 方面。你也能说，它是在实际的应用程序中，其中一个操作将使用 Spring AOP 框架。 |
|    Advice     | 这是实际行动之前或之后执行的方法。这是在程序执行期间通过 Spring AOP 框架实际被调用的代码。 |
|   Pointcut    | 这是一组一个或多个连接点，通知应该被执行。你可以使用表达式或模式指定切入点正如我们将在 AOP 的例子中看到的。 |
| Introduction  |           引用允许你添加新方法或属性到现有的类中。           |
| Target object | 被一个或者多个方面所通知的对象，这个对象永远是一个被代理对象。也称为被通知对象。 |
|    Weaving    | Weaving 把方面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时，类加载时和运行时完成。 |

​	关于通知的几个概念：

|      通知      |                             描述                             |
| :------------: | :----------------------------------------------------------: |
|    前置通知    |                在一个方法执行之前，执行通知。                |
|    后置通知    |         在一个方法执行之后，不考虑其结果，执行通知。         |
|   返回后通知   |   在一个方法执行之后，只有在方法成功完成时，才能执行通知。   |
| 抛出异常后通知 | 在一个方法执行之后，只有在方法退出抛出异常时，才能执行通知。 |
|    环绕通知    |             在建议方法调用之前和之后，执行通知。             |

### Spring事务管理

​	Spring 框架在不同的底层事务管理 APIs 的顶部提供了一个抽象层。Spring 的事务支持旨在通过添加事务能力到 POJOs 来提供给 EJB 事务一个选择方案。Spring 支持编程式和声明式事务管理。EJBs 需要一个应用程序服务器，但 Spring 事务管理可以在不需要应用程序服务器的情况下实现。同时，Spring直接抛出Exception就不会回滚，而抛出RuntimeException或其子类才会回滚

​	Spring事务管理的五大属性：隔离级别、传播行为、是否只读、事务超时、回滚规则。

​	隔离级别：

|          隔离级别          |                             含义                             |
| :------------------------: | :----------------------------------------------------------: |
|     ISOLATION_DEFAULT      | 使用底层数据存储的默认隔离级别。 所有其他级别对应于 JDBC 隔离级别。 |
| ISOLATION_READ_UNCOMMITTED | 表示可能发生脏读、不可重复读和幻读。<br/>此级别允许由一个事务更改的行在提交该行中的任何更改之前被另一个事务读取（“脏读”）。 如果任何更改被回滚，则第二个事务将检索到无效行。 |
|  ISOLATION_READ_COMMITTED  | 表示防止脏读； 可能发生不可重复读和幻读。<br/>此级别仅禁止事务读取其中包含未提交更改的行。 |
| ISOLATION_REPEATABLE_READ  | 表示防止脏读和不可重复读； 可能会发生幻读。<br/>此级别禁止事务读取未提交更改的行，也禁止一个事务读取一行，第二个事务更改该行，第一个事务重新读取该行，第二个获取不同值的情况时间（“不可重复读取”）。 |
|   ISOLATION_SERIALIZABLE   | 表示防止脏读、不可重复读和幻读。<br/>该级别包括ISOLATION_REPEATABLE_READ的禁止，并进一步禁止一个事务读取满足WHERE条件的所有行，第二个事务插入满足WHERE条件的行，第一个事务重新读取相同条件的情况，检索第二次阅读中的附加“幻影”行。 |

传播行为：传播行为定义关于客户端和被调用方法的事务边界。Spring定义了7中传播行为。

|         传播行为          |                             意义                             |
| :-----------------------: | :----------------------------------------------------------: |
|   PROPAGATION_REQUIRED    | 支持当前事务； 如果不存在，则创建一个新的。 类似于同名的 EJB 事务属性。<br>如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务 |
|   PROPAGATION_SUPPORTS    | 支持当前事务； 如果不存在则以非事务方式执行。 类似于同名的 EJB 事务属性。<br/>当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行 |
|   PROPAGATION_MANDATORY   | 支持当前事务； 如果当前事务不存在，则抛出异常。 类似于同名的 EJB 事务属性。<br/>当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。 |
| PROPAGATION_REQUIRES_NEW  | 创建一个新事务，如果存在则暂停当前事务。 类似于同名的 EJB 事务属性。<br/>创建一个新事务，如果存在当前事务，则挂起该事务。 |
| PROPAGATION_NOT_SUPPORTED | 不支持当前事务； 而是始终以非事务方式执行。 类似于同名的 EJB 事务属性。<br/>始终以非事务方式执行,如果当前存在事务，则挂起当前事务 |
|     PROPAGATION_NEVER     | 不支持当前事务； 如果当前事务存在则抛出异常。 类似于同名的 EJB 事务属性。<br/>不使用事务，如果当前事务存在，则抛出异常 |
|    PROPAGATION_NESTED     | 如果当前事务存在，则在嵌套事务中执行，否则行为类似于PROPAGATION_REQUIRED 。 EJB 中没有类似的特性。<br/>如果当前事务存在，则在嵌套事务中执行，否则REQUIRED的操作一样（开启一个事务） |

### Spring面试中问到的问题

* Spring启动的流程

​	

* Spring时如何解决循环依赖的

​	这里以AB两个bean为例，创建一个bean分为实例化和属性赋值，实例化后的bean还不能使用，属性赋值后才能使用，类似于声明和赋值。实例化A时，发现容器中没有A实例，会创建一个A实例，但是发现A依赖B，再递归去查找B，发现B也没有，于是创建B实例，又发现B依赖A，但这时候A已经有了一个实例化，虽然只是半成品，把A赋值给B里面后，返回递归，最后A的实例化完成。

* Spring中bean的生命周期

​	Spring中bean的生命周期主要包含四个阶段：实例化、属性赋值、初始化、销毁。实例化相当于声明，有三种实例化的方法，普通构造方法创建、静态工厂方法创建、实例化工厂方法创建。实例化和依赖注入的区别：实例化后的bean不是直接能使用的，还需要后续的初始化操作，依赖注入的bean是直接可以使用的。属性赋值阶段：主要是为bean的属性进行赋值，例如A依赖B，需要在这个时候将B注入到A中。然后就是初始化了：有 Aware 接口的依赖注入、BeanPostProcessor 在初始化前后的处理以及 InitializingBean 和 init-method 的初始化操作。然后初始化的bean就是可以使用的。最后就是销毁bean，通过DisposableBean 和 destory-method 。

* Spring中的三级缓存和解决循环依赖

​	Spring中的三级缓存：singletonObjects：一级缓存，主要存放初始化完成之后的对象。singletonFactories：三级缓存：存放的工厂对象，能够生成的实例化后的对象。earlySingletonObjects：相当于半成品：实例化完成，但是初始化没有完成的对象。这里以AB两个实例举例。A实例化后，发现需要依赖B，在一二级缓存中查找，发现没有找到，在三级缓存中调用工厂方法实例化，实例化B的过程中发现需要注入A，在二级缓存中找到，将其注入。返回到A到初始化，A初始化完成，放入一级缓存。需要注意，Spring中多例的循环依赖无法解决，会报错。

* Spring容器的启动流程

​	这一块的代码主要是在ApplicationContext里面，使用注解的话就是AnnotationConfigApplicationContext，使用xml的话就在ClassPathXmlApplicationContext。主要流程就是1）初始化Spring容器。2）将配置类的BeanDefinition注册到容器中。3）刷新容器。

### 缓存

Spring内部的：

//IOC - 控制反转

//IOC容器

//Spring 负责创建和管理对象 (Bean)的生命周期和配置（比较复杂，可以后期详细看看）

//Spring中bean的声明周期和作用域

//自动装配的原理

//阅读Spring两个容器相关的源码



//AOP - 面向切面编程

//Spring核心容器

//事务管理 - 提供了用于事务管理的通用抽象层

Spring是如何加载xml文件的（可能比较复杂，后续再看源码）



Spring中注解的含义

//Spring的事务



Mybatis相关的

JDBC 异常

JDBC和ORM

Spring如何使用jdbc连接的

Mybatis的原理和使用的逻辑



web

WEB和websocket

Spring中Servlet



SpringBoot和Spring的区别

SpringBoot的启动流程和原理





















