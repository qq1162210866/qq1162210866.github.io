## SpringBoot使用事务

​	上次了解了MySQL的事务概念，下面就开始编写代码来实际理解一下概念。

​	配置上面没有什么新加的配置，使用的数据库是MySQL，集成的Mybatis。按照之前的博客配置就可以了，这里不再累述。下面就直接上代码。然后说一下遇到的问题。

```java
package com.psq.train.mysql;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.support.SqlSessionDaoSupport;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;

/**
 * TransactionalTrain.java
 * Description: SpringBoot数据库事务练习
 *
 * @author Peng Shiquan
 * @date 2020/6/13
 */
@Service
public class TransactionalTrain extends SqlSessionDaoSupport {

    @Autowired
    private SqlSessionFactory sqlSessionFactory;

    @Override
    @Resource
    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
        super.setSqlSessionFactory(sqlSessionFactory);
    }


    //@PostConstruct
    public void mybatisTransactionalTrain() {
        //开启事务
        SqlSession sqlSession = sqlSessionFactory.openSession(false);
        TestUser testUser = new TestUser();
        testUser.setId(3);
        testUser.setName("qsp");
        testUser.setPassword("87654321");
        try {
            Integer saveResult = sqlSession.insert("com.psq.train.dao.UserMapper.insertUser", testUser);
            //重复插入，id相同一定回产生异常
            Integer saveResult2 = sqlSession.insert("com.psq.train.dao.UserMapper.insertUser", testUser);
            sqlSession.commit();
        } catch (Exception e) {
            System.err.println("事务开始回滚");
            sqlSession.rollback();
            System.err.println("事务回滚成功");
            throw e;
        } finally {
            sqlSession.close();
        }
    }
}
```

​	首先是`SqlSessionFactory`的注入，因为Mybatis取消了`SqlSessionFactory`的自动注入，所以这里需要继承一下`SqlSessionDaoSupport`，再重写一下`setSqlSessionFactory`方法。后面其他方法就可以使用一下代码正常注入了：

```java
@Autowired
private SqlSessionFactory sqlSessionFactory;
```

​	下面就是逻辑代码的堆叠了，需要注意的就是`sqlSessionFactory.openSession(false);`的意思是关闭自动提交，这样就可以测试事务了。然后就是插入两个主键相同的列，第一个可以正常插入，第二个因为逐渐相同会插入失败，导致异常，捕获异常后执行`sqlSession.rollback();`就可以回滚了。这里也有一个地方，执行`sqlSession.close();`的时候也会执行回滚，通过看源码的时候可以发现最后是执行了回滚操作的。

​	然后问题就来了，这里我尝试了很多次，事务回滚一直没有执行成功，数据库中最后仍然是一条数据。排查了很多的地方还是不行，这里也希望大神能够指导一下。

​	最后没有办法，只能通过Spring来实现事务。

​	Spring中的事务管理分为编程式事务管理和声明式事务管理。编程式事务管理现在使用的已经很少了，现在大部分都是声明式事务管理。我这里也是通过注解的形式来实现事务的管理。上代码。

```java
package com.psq.train.mysql;

import com.psq.train.dao.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

/**
 * SpringTransactionalTrain.java
 * Description:  使用Spring来管理事务
 *
 * @author Peng Shiquan
 * @date 2020/6/13
 */
@Component
public class SpringTransactionalTrain {

    @Autowired
    private UserMapper userMapper;

    @Transactional
    public void mybatisTransactionalTrain() {
        TestUser testUser = new TestUser();
        testUser.setId(3);
        testUser.setName("psq");
        testUser.setPassword("9876");
        System.err.println(testUser.toString());
        try {
            Integer saveResult = userMapper.insertUser(testUser);
            System.err.println("插入成功");
            //一定会报异常
            Integer saveResult2 = userMapper.insertUser(testUser);
        } catch (Exception e) {
            System.err.println(e);
            System.err.println("开始回滚");
            throw e;
        }

    }
}
```

​	通过controller层的调用，controller层代码如下：

```java
package com.psq.train.controller;

import com.psq.train.dao.UserMapper;
import com.psq.train.mysql.SpringTransactionalTrain;
import com.psq.train.mysql.TestUser;
import com.psq.train.mysql.TransactionalTrain;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

/**
 * TransactionalController.java
 * Description:  controller类，用于测试
 *
 * @author Peng Shiquan
 * @date 2020/6/13
 */
@RestController
public class TransactionalController {

    @Autowired
    private SpringTransactionalTrain springTransactionalTrain;
    

    @RequestMapping(value = "/user", method = RequestMethod.GET)
    public String saveTestUser() {
        springTransactionalTrain.mybatisTransactionalTrain();
        return "hello";
    }
}
```

​	按照上面的代码，事务就可以正常回滚。数据库中就没有新插入的记录。	

​	下面就简单说一下spring中的事务的一些信息。

* 事务的传播行为
  * propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是Spring默认的选择。
  * propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。
  * propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。
  * propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。
  * propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
  * propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。
  * propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作。
* 事务的实现方式
  * 编程式事务管理对基于 POJO 的应用来说是唯一选择。我们需要在代码中调用beginTransaction()、commit()、rollback()等事务管理相关的方法，这就是编程式事务管理。
  * 基于 TransactionProxyFactoryBean的声明式事务管理。
  * 基于 @Transactional 的声明式事务管理。
  * 基于Aspectj AOP配置事务。

