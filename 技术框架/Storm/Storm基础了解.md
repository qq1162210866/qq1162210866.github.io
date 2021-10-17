## Storm基础概念了解

​	最近公司需要学习Storm，所以就来了解一下。本次的博客也是了解一下基础的概念，不涉及代码和其他的东西。大部分的东西都是来自于官网。

​	Apache Storm是一个免费的开源分布式实时计算系统。通过Apache Storm，可以轻松可靠地处理无限制的数据流，从而可以进行实时处理，而Hadoop可以进行批处理。Apache Storm很简单，可以与任何编程语言一起使用，并且使用起来很有趣！

![image-20200706150422939](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200706150422939.png)

### Storm中的基础概念

* Topologies（拓扑）。实时应用程序的逻辑被封装在 Storm topology（拓扑）中. Storm topology（拓扑）类似于 MapReduce 作业.两者之间关键的区别是 MapReduce 作业最终会完成, 而 topology（拓扑）任务会永远运行（除非 kill 掉它）. 一个拓扑是 Spout 和 Bolt 通过 stream groupings 连接起来的有向无环图.
* Streams（流）。stream 是 Storm 中的核心概念.一个 stream 是一个无界的、以分布式方式并行创建和处理的 Tuple 序列. stream 以一个 schema 来定义, 这个 schema 用来命名 stream tuple（元组）中的字段.默认情况下 Tuple 可以包含 integers, longs, shorts, bytes, strings, doubles, floats, booleans, and byte arrays 等数据类型.你也可以定义自己的 serializers, 以至于可以在 Tuple 中使用自定义的类型.
* Spouts。Spout 是一个 topology（拓扑）中 streams 的源头. 通常 Spout 会从外部数据源读取 Tuple，然后把他们发送到拓扑中.
* Bolts。拓扑中所有的业务处理都在 Bolts 中完成. Bolt 可以做很多事情，过滤, 函数, 聚合, 关联, 与数据库交互等.
* Stream groupings。topology（拓扑）定义中有一部分是为每一个 bolt 指定输入的 streams . stream grouping 定义了stream 如何在 Bolts tasks 之间分区.相当于将流分组，内置有八个.
* Tasks。每个 Spout 或者 Bolt 都以跨集群的多个 Task 方式执行. 每个 Task 对应一个 execution 的线程.
* Workers。Topologies （拓扑）在一个或者跨多个 worker 执行. 每个 Worker 进程是一个物理的 JVM.

​	上面的信息都是来自于官网，官网地址为：[http://Storm.apache.org](http://Storm.apache.org)。

​	介绍完了一些基础概念，下面就说说具体的流程，把这些概念串起来。

​	Storm集群上有两种节点：主节点和工作节点。主节点运行一个名为“ Nimbus”的守护程序，该守护程序类似于Hadoop的“ JobTracker”。Nimbus负责在集群中分发代码，向计算机分配任务以及监视故障。

​	每个工作程序节点都运行一个称为“ Supervisor”的守护程序。主管侦听分配给它的机器的工作，并根据Nimbus分配给它的东西启动和停止必要的工作程序。每个工作进程执行一个拓扑子集；一个正在运行的拓扑由分布在许多计算机上的许多工作进程组成。

​	Nimbus和主管之间的所有协调都是通过Zookeeper集群完成的。此外，Nimbus守护程序和Supervisor守护程序是快速故障且无状态的。所有状态都保存在Zookeeper或本地磁盘中。

![image-20200706221725445](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200706221725445.png)

​	Spout是streams的源头。例如，Spout可能会从Kestrel队列中读取元组并将其作为流发出。否则，Spout可能会连接到Twitter API并发出一系列推文。

​	Bolts消耗任何数量的输入流，进行一些处理，并可能发出新的流。复杂的流转换（例如从一条推文流中计算趋势主题流）需要多个步骤，因此需要多个Bolts。Bolts可以执行任何功能，包括运行功能，过滤元组，进行流聚合，进行流联接，与数据库对话等等。

Spout和Bolts网络打包成一个“拓扑”，这是您提交给Storm集群以执行的顶级抽象。拓扑是流转换的图形，其中每个节点都是Spout或Bolts。

![image-20200706221938945](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200706221938945.png)

​	需要注意：Storm拓扑中的每个节点都并行执行。在拓扑中，您可以为每个节点指定所需的并行度，然后Storm将在整个集群中产生该数量的线程来执行。

​	拓扑将永远运行，或者直到您将其杀死为止。Storm将自动重新分配所有失败的任务。此外，Storm保证即使机器宕机和消息丢失也不会丢失数据。

​	剩下的就是具体编写代码去验证和深入了解了，说实话，看这些枯燥的概念很容易犯困，所以我的意见还是找一个demo，边写代码边理解，这样比较容易理解，同时也会有一个类似于奖励的机制，才能学的更快。

​	下一遍就要来写代码了。	

​	就这样吧，结束。