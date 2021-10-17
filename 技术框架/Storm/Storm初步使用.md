## Storm初步使用

​	上一篇博客简单的介绍了Strom的一个Demo，但是没有介绍使用中的细节，这篇博客就来介绍一下上次遗漏的细节。算是一个进阶的学习吧。开始。

### Strom分组策略

* **Shuffle grouping**: 随机分组，Tuples以一定方式随机分布在blot的task上，从而确保每个blot都具有相等数量的tuples。
* **Fields grouping**: 字段分组，stream按fields中指定的字段进行分组。例如，如果stream按照“ user-id”字段分组，则具有相同“ user-id”的Tuples将始终分到相同的task，但是具有不同“ user-id”的元组可能会到不同的task。
* **Partial Key grouping**: 部分密钥分组，stream按照指定的字段进行分组（如同上面的字段分组策略），但在两个下游blot之间进行负载平衡，当输入数据倾斜时，可以更好地利用资源。官方文档对它的工作原理和优点提供了很好的解释。地址如下：[官网](https://melmeric.files.wordpress.com/2014/11/the-power-of-both-choices-practical-load-balancing-for-distributed-stream-processing-engines.pdf)
* **All grouping**: 所有分组，stream在blot的所有task之间复制。请谨慎使用此分组策略。
* **Global grouping**: 全局分组，整个stream进入blot的task的其中一项。具体来说，它将转到ID最低的task上。
* **None grouping**:无分组，此分组指定您不关心stream的分组方式。没有分组策略等效于随机分组策略。不过，最终Storm会向下推没有分组的blot，最后让其订阅的blot或spout在同一线程中执行（如果可能）。（翻译就是这样，感觉还是不清楚细节，但是类似于随机分组）
* **Direct grouping**:直接分组，这是一种特殊的分组。以这种方式分组意味着Tuples的**生产者**决定消费者的哪个task将接收此Tuples。直接分组只能在已经声明为直接流的流上声明。直接流发出的元组必须使用`emitDirect`方法之一发出。blot可以使用提供的`TopologyContext`或通过`emit`在`OutputCollector`中跟踪方法的输出（返回处理Tuples的task的ID）来获取其使用者的任务ID。
* **Local or shuffle grouping**: 本地或随机分组，如果目标blot在同一工作进程中具有一个或多个任务，则元组将被随机转换为那些正在进行的任务。否则，这就像常规的随机分组。

​	讲完了上面的八种分组策略，下面就来看看之前的demo中是如何使用的。

​	`fieldsGrouping("ExampleSpout", new Fields("word"));`这段代码表示的是从实体`ExampleSpout`出来的ID为`default`的流按照字段分组，字段`word`内容相同的`tuple`将会交给相同的`task`处理。如下图：

![image-20200904174302403](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200904174302403.png)

​	在blot中声明出去的流时，一个blot可能会发送出多个格式的流。同时，也可以指定流的ID，方便下游的多个blot接收。如下图：

![image-20200904174842732](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200904174842732.png)

## Storm中并行度理解

​	Storm中有一个并行度的概念，在设置blot的时候也可以设置这个东西，下面就来说说这个概念和它牵扯的一下东西。

​	了解并行度需要先了解Strom的三个实体的概念。

* Worker processes。一个工作进程执行拓扑的一个子集。辅助进程属于特定的拓扑，并且可以为该拓扑的一个或多个组件（blot或spout）运行一个或多个执行程序。正在运行的拓扑就是由在Storm集群中，许多计算机上运行的许多此类进程组成。
* Executors (threads)。一个执行者是由一个工作进程催生了的一个线程。它可以为同一组件（blot或spout）运行一项或多项任务。
* Tasks。一个任务是执行实际的数据处理的最小单元。您在代码中实现的每个spout或blot在整个集群中执行的任务数量一样多。在拓扑的整个生命周期中，组件的任务数量始终是相同的，但是组件的执行程序（线程）的数量会随着时间而变化。这意味着以下条件成立：threads ≤ tasks。默认情况下，任务数设置为与执行程序数相同，即Storm将在每个线程中运行一个任务。

​	他们的关系如下图：

![image-20200906234133843](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200906234133843.png)

​	详细的解析也可以看这个博客：[https://www.jianshu.com/p/ae619304d881](https://www.jianshu.com/p/ae619304d881)这个博客将的比较细，同时还有一些自己想法后的测试。

​	当然，也可以看官网上的解析，这些都是从官网了解到的，官网地址如下：[http://storm.apache.org/releases/2.2.0/Understanding-the-parallelism-of-a-Storm-topology.html](http://storm.apache.org/releases/2.2.0/Understanding-the-parallelism-of-a-Storm-topology.html)。

​	了解了这些基础概念，就需要引入一个demo，开始了解并行度。代码如下：

```java
Config conf = new Config();
conf.setNumWorkers(2); // use two worker processes

topologyBuilder.setSpout("blue-spout", new BlueSpout(), 2); // set parallelism hint to 2

topologyBuilder.setBolt("green-bolt", new GreenBolt(), 2)
               .setNumTasks(4)
               .shuffleGrouping("blue-spout");

topologyBuilder.setBolt("yellow-bolt", new YellowBolt(), 6)
               .shuffleGrouping("green-bolt");

StormSubmitter.submitTopology(
        "mytopology",
        conf,
        topologyBuilder.createTopology()
    );
```

​	代码很好理解，spout和第一个blot的parallelism hint为2，最后的blot则为6。则得出这段程序总的并行度为：（2+2+6）/2=5至于原因上面的博客和官网也有解释。下面的图也是讲解：

![image-20200907000012034](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200907000012034.png)

​	因为config设置了两个worker，但是总的线程有10个，相除即得并行度为5。但是这里还是不清楚并行度为5的意义和参考价值。可能是一个worker里面有多少executor吧。

​	后续公司也会把Strom的框架换掉，后续很有可能不再更新Strom的博客了。没有深入了解很是遗憾，但是人生不就是这样吧，终会有点不完美。事在人为有时就是说说而已，很多事我们没法决定和左右，只能尽自己最大的努力。

​	就这样吧，结束。