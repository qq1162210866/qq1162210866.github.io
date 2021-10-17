## SpringBoot知识点

​	Spring Boot 是 Spring 开源组织下的子项目，是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。

### SpringBoot的启动流程

​	流程图如下：

​	![image-20210623161300236](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210623161300236.png)

​	流程暂时还不是很清楚，可以后续完善

### SpringBoot中的自动配置原理

​	项目中只要配置了redis的信息，在代码中直接注入redis模版bean，就可以直接使用了，这个就是自动配置。将配置文件中的信息配置到bean上，自动配置分为bean的自动配置和属性的自动配置。

​	其中`@EnableAutoConfiguration`是关键(启用自动配置)，内部实际上就去加载`META-INF/spring.factories`文件的信息，然后筛选出以`EnableAutoConfiguration`为key的数据，加载到IOC容器中，实现自动配置功能！

### Spring JavaConfig

​	因为xml配置bean的方式比较复杂，并且不符合面向对象的逻辑。所以就有了Spring JavaConfig，它允许开发者将bean定义和在Spring配置XML文件到Java类中。这两者是可以混用的。

### SpringBoot中配置文件加载顺序

​	spring boot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件。properties优先级会高一些，相同名称的配置，会将 yml内的配置覆盖掉。

```xml
–file:./config/
–file:./
–classpath:/config/
–classpath:/
```

​	顺序如下，由一到四：

![image-20210623170034583](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210623170034583.png)

注意：bootstrap 是应用程序的父上下文，也就是说 bootstrap 加载优先于 applicaton。bootstarp是spring cloud中使用的配置文件。

### YAML文件

​	YAML 的语法和其他高级语言类似，并且可以简单表达清单、散列表，标量等数据形态。它使用空白符号缩进和大量依赖外观的特色，特别适合用来表达或编辑数据结构、各种配置文件、倾印调试内容、文件大纲（例如：许多电子邮件标题格式和YAML非常接近）。

​	YAML 的配置文件后缀为 **.yml**，如：**runoob.yml** 。

### SpringBoot中的监听器

​	监听器模式是由观察者模式发展而来，有一下几个重要部分。

* 事件：步骤1和步骤2，通过对天气进行抽象，并实现下雨和下雪的天气状态
* 监听器：步骤3和步骤4，规范对天气监听的模式，并且规范对应天气下，需要如何处理
* 广播器：步骤5、步骤6和步骤7，当有事件发生的时候，广播器发出信号，告知所有的监听器，监听器根据事件作出相应的处理。触发下雨事件的时候，下雨监听器收到消息，它抬头一看乌云密布电闪雷鸣，微微一愣，大喊一句：“打雷下雨收衣服啊！！”，广播器继续通知下一个监听器下雪监听器，下雪监听器看看天空，摆摆手，说：“这事与我无关去找别人”
* 触发机制：步骤8，demo中采用的硬编码的形式触发的，在实际运用中，可能是湿度仪检测到湿度暴涨开始下雨了，触发广播。

​	具体的流程就是：监听器中编写对某些事件发生的处理情况，然后监听器注册到广播器中，当有时间触发时，广播器告知所有的监听器，监听器在做相应的处理。

​	ApplicationListener是Spring事件机制的一部分，与抽象类ApplicationEvent类配合来完成ApplicationContext的事件机制，实现ApplicationListener接口的类，会在SpringBoot加入到广播器中，当ApplicationContext触发了一个事件，就用广播器通知所有实现ApplicationListener接口的类。

​	SpringBoot中九大事件：

* EventObject：事件顶级对象，所有事件对象的根对象
* ApplicationEvent：应用事件
* SpringApplicationEvent：Spring自己的事件，Spring框架自身的事件都会实现这个接口
* ApplicationStartingEvent：启动事件，框架刚刚启动就会发出这个事件
* ApplicationEnvironmentPreparedEvent：环境在变完成，系统属性和用户指定已经加载完成
* ApplicationContextInitializedEvent：已经创建好了上下文，并且还没有加载任何bean之前发出这个事件
* ApplicationPreparedEvent：在Bean定义开始加载之后，尚未完全加载之前，刷新上下文之前触发
* ApplicationStartedEvent：bean已经创建完成，上下文已经刷新完成，但是ApplicationRunner和CommandLineRunne两个扩展接口并未执行
* ApplicationReadyEvent：ApplicationRunner和CommandLineRunne两个扩展接口执行完成之后触发
* ApplicationFailedEvent：在启动发生异常时触发

​	启动 —》ApplicationStartingEvent —》ApplicationEnvironmentPreparedEvent —》ApplicationContextInitializedEvent —》 ApplicationPreparedEvent —》ApplicationStartedEvent —》 ApplicationReadyEvent —》启动完毕

### Spring Boot 中的 starter 

​	在spring中引入一个中间件或者JPA，需要先在pom文件中导入需要的依赖，再添加相应的配置。然后再调试完毕，直到项目正常启动。多个项目也是这样的配置，只不过多个项目可能配置直接复制即可，本质还是复杂的。

​	starter的主要目的就是为了解决上面的这些问题。

　　starter的理念：starter会把所有用到的依赖都给包含进来，避免了开发者自己去引入依赖所带来的麻烦。需要注意的是不同的starter是为了解决不同的依赖，所以它们内部的实现可能会有很大的差异，例如jpa的starter和Redis的starter可能实现就不一样，这是因为starter的本质在于synthesize，这是一层在逻辑层面的抽象，也许这种理念有点类似于Docker，因为它们都是在做一个“包装”的操作，如果你知道Docker是为了解决什么问题的，也许你可以用Docker和starter做一个类比。

　　starter的实现：虽然不同的starter实现起来各有差异，但是他们基本上都会使用到两个相同的内容：ConfigurationProperties和AutoConfiguration。因为Spring Boot坚信“约定大于配置”这一理念，所以我们使用ConfigurationProperties来保存我们的配置，并且这些配置都可以有一个默认值，即在我们没有主动覆写原始配置的情况下，默认值就会生效，这在很多情况下是非常有用的。除此之外，starter的ConfigurationProperties还使得所有的配置属性被聚集到一个文件中（一般在resources目录下的application.properties），这样我们就告别了Spring项目中XML地狱。

​	对于starter中的配置，用户仍然可以再application配置文件中进行覆盖掉，达到自己的定制化。没有覆盖的就使用starter中的默认值。

### Spring Security 和 Shiro

​	Spring Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI（控制反转Inversion of Control ,DI:Dependency Injection 依赖注入）和AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。

​	Apache Shiro 是一个开源安全框架，提供身份验证、授权、密码学和会话管理。Shiro 框架具有直观、易用等特性，同时也能提供健壮的安全性，虽然它的功能不如 Spring Security 那么强大，但是在常规的企业级应用中，其实也够用了。

​	两者都是权限框架，用来做安全验证的，考点估计也比较少。后续遇到再注意。

### WebSockets

​	WebSocket是一种网络传输协议，可在单个TCP连接上进行全双工通信，位于OSI模型的应用层。WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就可以创建持久性的连接，并进行双向数据传输。

​	后续如果有人问的话，可以看看这篇文章：https://www.cnblogs.com/chyingp/p/websocket-deep-in.html

### 日志框架

​	在Java开发中，常用的日志记录框架有JDKLog、Log4J、LogBack、SLF4J、SLF4J。这些日志记录框架各有各的特点，各有各的应用场景。

​	日志框架使用的是门面模式，分为门面和实现。JCL、SLF4J是日志门面。JUL、LOG4J1、LOG4J2、LOGBACK是日志实现框架。

​	common-logging 与 slf4j 都是日志实现门面，它们本身都不具体实现写日记操作，只是提供统一的接口。

​	common-logging 是最早的日志门面实现，而 SLF4J 是较新的日志门面实现。因此 SLF4J 在性能以及兼容性处理会比 common-logging要好。

* JDKLog：jdk自带的日志，直接就可以使用。但 JDKLog 功能比较太过于简单，不支持占位符显示，拓展性比较差，所以现在用的人也很少。
* Log4J： 是 Apache 的一个日志开源框架，有多个分级（DEBUG/INFO/WARN/ERROR）记录级别，可以很好地将不同日志级别的日志分开记录，极大地方便了日志的查看。
* LogBack ：其实可以说是 Log4J 的进化版，因为它们两个都是同一个人（Ceki Gülcü）设计的开源日志组件。LogBack 除了具备 Log4j 的所有优点之外，还解决了 Log4J 不能使用占位符的问题。

### spring.factories

#### SPI机制

​	SPI的全名为Service Provider Interface.我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块的方案，xml解析模块、jdbc模块的方案等。面向的对象的设计里，我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。 
​	java SPI就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。

​	在Spring中也有一种类似与Java SPI的加载机制。它在META-INF/spring.factories文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。 这种自定义的SPI机制是Spring Boot Starter实现的基础。 

### SpringBoot中常用的注解

* @ComponentScan主要就是定义扫描的路径从中找出标识了需要装配的类自动装配到spring的bean容器中。常和@Component搭配使用。
* @Component：表示带注释的类是“组件”。 在使用基于注释的配置和类路径扫描时，此类类被视为自动检测的候选对象。
* @Autowired:默认按类型匹配注入Bean.如果找到两个，就会报异常，因为没法判断到底使用哪个
* @Resource:默认按名称匹配注入Bean，先是按照名称查询bean，如果没有找到按照类型，多个也会报错。
* @Qualifier:指定注入bean的名称，需要搭配@Autowired一起使用。

### SpringBoot面试问题

* SpringBoot启动流程

​	第一部分进行SpringApplication的初始化模块，配置一些基本的环境变量、资源、构造器、监听器，第二部分实现了应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块，第三部分是自动化配置模块。背不住

​	

### 缓存

//springboot自动配置原理

//javaconfig

//@ComponentScan扫描

//springboot配置加载顺序

//springboot监听器

//Spring Boot 中的 starter 

//yaml文件

//Spring Security 和 Shiro的区别和作用

//websockets

//日志框架

//spring.factories



需要练习的有：









