## Storm简单demo

​	上一篇介绍了Storm的基础概念，下面就来实际写个demo。

​	因为是开发，所以基础的配置还是没有做，就是说是在本地跑的，没有集群之类的。

​	先整合一个基础的架子，使用的是SpringBoot的框架，但是移除了基础依赖，添加了Storm的依赖。上代码	前还是需要先解释一下一些基础的概念。然后再上代码。

​	Storm中使用tuple来作为数据元，相当于一个一个水滴，很多水滴汇集成流。每个tuple是一堆值，每个值有一个名字，并且每个值可以是任何类型。tuple中有两个概念，Values和Fields。tuple也是由这两个组成。

* Values类直接继承自Java中的ArrayList类，因为ArrayList类恰好能够很好地满足描述“一行”值的需要——有序、不去重、可伸缩。

* Fields是为了定义Values的名字。在使用Values描述了一行值的概念之后，接下来如何知道这行值当中每一列值代表的含义，也就是要知道字段声明，就是Fields。

​	知道这些概念后，剩下的就是代码的流程了。

​	代码首先新建一个拓扑，然后往拓扑中放入数据源Spout，在依次放入处理单元Bolt，根据需要对这些组件进行设置，比如数据的分组策略和线程的个数等等。Bolt有两个，第一个是先对数据进行初步处理，第二个Bolt对处理后的数据进行统计。最后在销毁拓扑的时候打印出统计的结果。流程就是这个样子，上代码。

​	Spout代码：

```java
package com.example.strom.spout;

import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;

import java.util.Map;

/**
 * ExampleSpout.java
 * Description: 练习Spout，数据当源头。也叫壶嘴
 *
 * @author Peng Shiquan
 * @date 2020/7/6
 */
public class ExampleSpout extends BaseRichSpout {

    /**
     * Spout输出收集器
     */
    private SpoutOutputCollector spo;
    /**
     *
     */
    private static final String filed = "word";
    /**
     * 计数变量
     */
    private int count = 1;
    /**
     * 用数组模拟流
     */
    private String[] messages = {"My nickname is xuwujing",
            "My blog address is http://www.panchengming.com/",
            "My interest is playing games"
    };


    /**
     * Description: 在spout组件初始化时被调用
     *
     * @param map
     * @param topologyContext
     * @param spoutOutputCollector
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/6
     */
    @Override
    public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {
        System.err.println("开始调用ExampleSpout的open方法");
        this.spo = spoutOutputCollector;
    }

    /**
     * Description: spout的核心，主要执行方法，用于输出信息
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/6
     */
    @Override
    public void nextTuple() {
        if (count <= messages.length) {
            System.err.println("第" + count + "次模拟发送数据...");
            /**
             * Values类所作的优化主要是提供若干个支持可变列表的构造方法，包括全为String的参数类型、全为Integer的参数类型以及通用的Object类型
             * Values类直接继承自Java中的ArrayList类，因为ArrayList类恰好能够很好地满足描述“一行”值的需要——有序、不去重、可伸缩。
             */
            /**
             * 必须设置messageId，才能调用ack方法
             */
            this.spo.emit(new Values(messages[count - 1]), count);
        }
        count++;
    }

    /**
     * Description: 声明数据格式，即输出的tuple中，包含了几个字段
     *
     * @param outputFieldsDeclarer
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        System.err.println("ExampleSpout开始声明数据格式");
        /**
         * 这里Fields声明了几个字段，传tuple就要传多少个Values
         */
        outputFieldsDeclarer.declare(new Fields(filed));
    }

    /**
     * Description: 当Topology停止时，会调用这个方法
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void close() {
        System.err.println("Topology停止");
        super.close();
    }

    /**
     * Description: 处理成功时，会调用这个方法
     *
     * @param msgId
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void ack(Object msgId) {
        System.err.println("消息处理成功，消息ID" + msgId);
        super.ack(msgId);
    }

    /**
     * Description: 当tuple处理失败时，会调用这个方法
     *
     * @param msgId
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void fail(Object msgId) {
        System.err.println("消息处理失败");
        super.fail(msgId);
    }
}
```

​	第一个Bolt代码：

```java
package com.example.strom.bolt;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

import java.util.Map;

/**
 * ExampleBolt.java
 * Description:  Bolt练习，处理流数据的地方
 *
 * @author Peng Shiquan
 * @date 2020/7/7
 */
public class SegmentationBolt extends BaseRichBolt {

    private OutputCollector outputCollector;

    /**
     * Description: 在Bolt启动前执行，提供启动的环境。
     *
     * @param map
     * @param topologyContext
     * @param outputCollector
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        System.err.println("进入SegmentationBolt的prepare");
        this.outputCollector = outputCollector;
    }

    /**
     * Description: Bolt的核心,执行的方法。每次接收一个tuple，就会调用一次
     *
     * @param tuple
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void execute(Tuple tuple) {
        System.err.println("开始执行处理方法");
        /**
         * Spout已经定义了这个field
         */
        String msg = tuple.getStringByField("word");
        String[] words = msg.toLowerCase().split(" ");

        for (String word : words) {
            /**
             * 发送到下个bolt
             */
            this.outputCollector.emit(new Values(word));
        }
        outputCollector.ack(tuple);
    }


    /**
     * Description: 声明数据格式，即输出的tuple中，包含了几个字段。Bolt执行完毕也会输出一个流
     *
     * @param outputFieldsDeclarer
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        System.err.println("ExampleSpout开始声明数据格式");
        outputFieldsDeclarer.declare(new Fields("count"));
    }

    /**
     * Description: 释放该Bolt占用的资源，Strom在终止时会调用这个方法
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void cleanup() {
        System.err.println("SegmentationBolt资源释放");
        super.cleanup();
    }
}
```

​	第二个Bolt代码：

```java
package com.example.strom.bolt;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.tuple.Tuple;

import java.util.HashMap;
import java.util.Map;

/**
 * CountBolt.java
 * Description: Bolt练习，计数的Bolt
 *
 * @author Peng Shiquan
 * @date 2020/7/7
 */
public class CountBolt extends BaseRichBolt {

    /**
     * 计数
     */
    private long count;
    /**
     * 保存单词和对应的计数
     */
    private HashMap<String, Integer> counts = null;

    @Override
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        System.err.println("进入CountBolt的prepare");
        this.counts = new HashMap<String, Integer>();
    }

    @Override
    public void execute(Tuple tuple) {
        String msg = tuple.getStringByField("count");
        System.err.println("第" + count + "次统计单词次数");
        if (!counts.containsKey(msg)) {
            counts.put(msg, 1);
        } else {
            counts.put(msg, counts.get(msg) + 1);
        }
        count++;
    }

    /**
     * Description: 最后执行，打印统计的单词次数
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void cleanup() {
        System.err.println("下面就是单词次数================");
        for (Map.Entry<String, Integer> entry : counts.entrySet()) {
            System.err.println("单词" + entry.getKey() + "出现" + entry.getValue() + "次");
        }
        System.err.println("输出结束=======================");
        System.err.println("释放资源");
        super.cleanup();
    }

    /**
     * Description: 不在往下一个Bolt输出，所以没有代码
     *
     * @param outputFieldsDeclarer
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        System.err.println("CountBolt开始声明数据格式");
    }
}
```

​	启动入口代码：

```java
package com.example.strom.bolt;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.tuple.Tuple;

import java.util.HashMap;
import java.util.Map;

/**
 * CountBolt.java
 * Description: Bolt练习，计数的Bolt
 *
 * @author Peng Shiquan
 * @date 2020/7/7
 */
public class CountBolt extends BaseRichBolt {

    /**
     * 计数
     */
    private long count;
    /**
     * 保存单词和对应的计数
     */
    private HashMap<String, Integer> counts = null;

    @Override
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        System.err.println("进入CountBolt的prepare");
        this.counts = new HashMap<String, Integer>();
    }

    @Override
    public void execute(Tuple tuple) {
        String msg = tuple.getStringByField("count");
        System.err.println("第" + count + "次统计单词次数");
        if (!counts.containsKey(msg)) {
            counts.put(msg, 1);
        } else {
            counts.put(msg, counts.get(msg) + 1);
        }
        count++;
    }

    /**
     * Description: 最后执行，打印统计的单词次数
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void cleanup() {
        System.err.println("下面就是单词次数================");
        for (Map.Entry<String, Integer> entry : counts.entrySet()) {
            System.err.println("单词" + entry.getKey() + "出现" + entry.getValue() + "次");
        }
        System.err.println("输出结束=======================");
        System.err.println("释放资源");
        super.cleanup();
    }

    /**
     * Description: 不在往下一个Bolt输出，所以没有代码
     *
     * @param outputFieldsDeclarer
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/7/7
     */
    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        System.err.println("CountBolt开始声明数据格式");
    }
}
```

​	剩下就没有什么主要代码了，还有就是pom文件和一些配置文件。因为没有使用SpringBoot的基础类，所以没有Spring的配置文件，只有一个日志文件是为了方便调试。配置文件代码如下。

pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.3.1.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
   </parent>
   <groupId>com.example</groupId>
   <artifactId>strom</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>strom</name>
   <description>Demo project for Spring Boot</description>

   <properties>
      <java.version>1.8</java.version>
   </properties>

   <dependencies>
      <!--storm相关jar  -->
      <dependency>
         <groupId>org.apache.storm</groupId>
         <artifactId>storm-core</artifactId>
         <version>1.1.1</version>
         <scope>compile</scope>
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

   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>

</project>
```

​	日志配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration monitorInterval="60">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%-4r [%t] %-5p %c{1.} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="org.apache.storm" level="OFF"/>
        <Logger name="org.apache.zookeeper" level="OFF"/>
        <Root level="OFF">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</configuration>
```

​	以上就是代码了，因为是简单的demo，只是本地模式，所以运行起来也很简单。后续的集群部署还需要在了解一下。运行的截图如下：

![image-20200708212602792](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200708212602792.png)

​	没有什么可以讲的了，后面就是要再写一个集群部署的知识点。

​	就这样吧，结束。

