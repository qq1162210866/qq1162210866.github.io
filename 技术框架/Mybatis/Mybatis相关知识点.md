## MyBatis

### ORM框架

​	对象-关系映射（Object-Relational Mapping，简称ORM），面向对象的开发方法是当今企业级应用开发环境中的主流开发方法，关系数据库是企业级应用环境中永久存放数据的主流数据存储系统。对象和关系数据是业务实体的两种表现形式，业务实体在内存中表现为对象，在数据库中表现为关系数据。内存中的对象之间存在关联和继承关系，而在数据库中，关系数据无法直接表达多对多关联和继承关系。因此，对象-关系映射(ORM)系统一般以中间件的形式存在，主要实现程序对象到关系数据库数据的映射。

​	MySQL中存储的是关系型数据，但是项目中使用的是对象型数据。ORM框架就是这两种数据的中间层，将两种数据进行转换。这也是它要做的事。

#### Hibernate

​	Hibernate 框架是一个全表映射的框架。通常开发者只要定义好持久化对象到数据库表的映射关系，就可以通过 Hibernate 框架提供的方法完成持久层操作。

​	相较于MyBatis，Hibernate是全自动的ORM框架，有它的优点：开发效率快，不需要开发人员懂SQL。但是也有着缺点：对于多表联合复杂查询支持较差，很难对SQL进行优化，更新的时候需要传全部的字段。

#### MyBatis

​	MyBatis 框架是一个半自动映射的框架。这里所谓的“半自动”是相对于 Hibernate 框架全表映射而言的，MyBatis 框架需要手动匹配提供 POJO、SQL 和映射关系，而 Hibernate 框架只需提供 POJO 和映射关系即可。

​	缺点：需要写SQL，开发效率相对于Hibernate较慢。对于互联网来说，SQL的执行效率需要优化，同时查询的复杂度也比较高。MyBatis就成了互联网行业比较好的选择。

### Spring连接JDBC的基本步骤

​	因为JDBC是面向接口编程，所以很多的驱动厂商已经写好了。只需要加载对应的驱动，就可以获取对应的数据库连接。Sping中是通过DataSource来获取数据库连接的。但是spring没有局限于此，又提供了JdbcTemplate这么一个类给我们使用！它封装了DataSource，也就是说我们可以在Dao中使用JdbcTemplate就行了。JdbcTemplate返回的是map，它并不知道我们需要的是什么类型的数据。

### MyBatis流程

​	工作原理图：

![image-20210622110302194](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210622110302194.png)

​	读取mybatisxml配置文件，读取数据库信息和mybatis设置信息。然后再读取映射xml文件。再通过Configuration对象构建SqlSessionFactory，再通过SqlSessionFactory创建SqlSession 对象，该对象中包含了执行 SQL 语句的所有方法。需要查询时，通过Executor 执行器，它将根据 SqlSession 传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。从数据库中查询出数据后，再通过MappedStatement对象将数据进行封装，该对象是对映射信息的封装，用于存储要映射的 SQL 语句的 id、参数等信息。	

* 建造者模式是日常开发中比较常见的设计模式，它的主要作用就是将复杂事物创建的过程抽象出来，该抽象的不同实现方式不同，创建出的对象也不同。通俗的讲，创建一个对象一般都会有一个固定的步骤，这个固定的步骤我们把它抽象出来，每个抽象步骤都会有不同的实现方式，不同的实现方式创建出的对象也将不同。

### Mybatis中的执行器（Executor）

​	Mybatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

* SimpleExecutor：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。
* ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。
* BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

### 延迟加载

​	延迟加载就是在查询一对多关系时，延迟加载一所对应的表中的信息，只有在需要一的信息时，才回去执行SQL查询一的信息。

​	它的原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

### Mybatis如何将xml文件映射为对象

​	通过mybatis的配置信息生成一个Configuration实例，然后Configuration实例读取mapper文件，来源分为xml文件和java注解。将SQL信息解析为MappedStatement对象，这个对象包含了传入参数映射配置、执行的SQL语句、结果映射配置。用户如果想要调用一条请求时，Mybatis会根据SQL的ID找到对应的MappedStatement对象，然后将请求的参数传入，得到解析后的SQL，拿着这条SQL在数据库中执行，得到结果后，将结果按照配置的映射关系转换，转换成java对象，完成一次请求。

​	Dao接口，就是人们常说的Mapper接口，接口的全限名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement，举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面id = findStudentById的MappedStatement。在Mybatis中，每一个<select>、<insert>、<update>、<delete>标签，都会被解析为一个MappedStatement对象。

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。

### #和$的区别，和原理

#{}是占位符，预编译处理；${}是拼接符，字符串替换，没有预编译处理。#{} 可以有效的防止SQL注入，提高系统安全性；${} 不能防止SQL 注入

Mybatis在处理#{}时，#{}传入参数是以字符串传入，会将SQL中的#{}替换为?号，调用PreparedStatement的set方法来赋值。

Mybatis在处理${}时，是原值传入，就是把${}替换成变量的值，相当于JDBC中的Statement编译。

变量替换后，#{} 对应的变量自动加上单引号 ‘’；变量替换后，${} 对应的变量不会加上单引号 ‘’

### Mybatis中的一级缓存和二级缓存

​	每个SqlSession中持有了Executor，每个Executor中有一个LocalCache。当用户发起查询时，MyBatis根据当前执行的语句生成MappedStatement，在Local Cache进行查询，如果缓存命中的话，直接返回结果给用户，如果缓存没有命中的话，查询数据库，结果写入Local Cache，最后返回结果给用户。

默认的是一级缓存，通过设置Mybatis的配置文件即可。共有两个选项，`SESSION`或者`STATEMENT`，默认是`SESSION`级别，即在一个MyBatis会话中执行的所有语句，都会共享这一个缓存。一种是`STATEMENT`级别，可以理解为缓存只对当前执行的这一个`Statement`有效。建议配置成STATEMENT级别。

​	在同一个方法里面，两个不同的SqlSession分别执行查询和修改，可能会造成脏读的情况。

注意：

* MyBatis一级缓存的生命周期和SqlSession一致。
* MyBatis一级缓存内部设计简单，只是一个没有容量限定的HashMap，在缓存的功能性上有所欠缺。
* MyBatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。

​	如果多个SqlSession之间需要共享缓存，则需要使用到二级缓存。开启二级缓存后，会使用CachingExecutor装饰Executor，进入一级缓存的查询流程前，先在CachingExecutor进行二级缓存的查询二级缓存开启后，同一个namespace下的所有操作语句，都影响着同一个Cache，即二级缓存被多个SqlSession共享，是一个全局的变量。

当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

注意：

* MyBatis的二级缓存相对于一级缓存来说，实现了`SqlSession`之间缓存数据的共享，同时粒度更加的细，能够到`namespace`级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。
* MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
* 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将MyBatis的Cache接口实现，有一定的开发成本，直接使用Redis、Memcached等分布式缓存可能成本更低，安全性也更高。

### Mybatis的核心组件

#### SqlSessionFactory

​	SqlSessionFactory是一个接口，主要的作用就是初始化Mybatis框架和提供SqlSession对象。实现类主要有两个：一个是 SqlSessionManager 类，一个是 DefaultSqlSessionFactory 类。

* DefaultSqlSessionFactory : SqlSessionFactory 的默认实现类，是真正生产会话的工厂类，这个类的实例的生命周期是全局的，它只会在首次调用时生成一个实例（单例模式），就一直存在直到服务器关闭。
* SqlSessionManager ： 已被废弃，原因大概是: SqlSessionManager 中需要维护一个自己的线程池，而使用MyBatis 更多的是要与 Spring 进行集成，并不会单独使用，所以维护自己的 ThreadLocal 并没有什么意义，所以 SqlSessionManager 已经不再使用。

​	流程就是：SqlSessionFactoryBuilder根据Mybatis配置的xml文件通过建造者模式的设计思想构建一个SqlSessionFactory对象。解析完成后返回一个 `DefaultSqlSessionFactory`对象，它是 SqlSessionFactory 的默认实现类。

#### SqlSession

​	SqlSessionFactory.openSession 过程中我们可以看到，会调用到 DefaultSqlSessionFactory 中的 openSessionFromDataSource 方法，这个方法主要创建了两个与我们分析执行流程重要的对象，一个是 Executor 执行器对象，一个是 SqlSession 对象。

​	SqlSession 对象是 MyBatis 中最重要的一个对象，这个接口能够让你执行命令，获取映射，管理事务。SqlSession 中定义了一系列模版方法，让你能够执行简单的 CRUD 操作，也可以通过 getMapper 获取 Mapper 层，执行自定义 SQL 语句，因为 SqlSession 在执行 SQL 语句之前是需要先开启一个会话，涉及到事务操作，所以还会有 commit、 rollback、close 等方法。这也是模版设计模式的一种应用。

#### Executor

​	每一个 SqlSession 都会拥有一个 Executor 对象，这个对象负责增删改查的具体操作，我们可以简单的将它理解为 JDBC 中 Statement 的封装版。 也可以理解为 SQL 的执行引擎，要干活总得有一个发起人吧，可以把 Executor 理解为发起人的角色。

​	Executor 执行器，它有两个实现类，分别是BaseExecutor和 CachingExecutor。

#### StatementHandler

​	`StatementHandler` 是四大组件中最重要的一个对象，负责操作 Statement 对象与数据库进行交互，在工作时还会使用 `ParameterHandler` 和 `ResultSetHandler`对参数进行映射，对结果进行实体类的绑定。


### 缓存

//ORM框架

//Spring连接JDBC的过程和步骤

//MyBatis如何将xml映射为对象

//statement是什么

​	Statement对象表示一个原始语句，其中将单个方法应用于目标和一组参数

//#和$的区别，和原理

//延迟加载

//一级缓存和二级缓存

//Mybatis的四个核心组件

//Mybatis中的执行器（Executor）