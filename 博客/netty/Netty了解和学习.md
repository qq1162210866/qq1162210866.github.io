
​	工作中需要集成一个TCP客户端，使用Java原生的`ServerSocket`写了一个demo，但是整合到项目中的时候发现原来的代码使用的是`Netty`，由于自己不是很了解，所以这次就来学习一下，首先按照惯例先了解Netty的基础概念。开始。

​	Netty是由JBOSS提供的一个java开源框架，现为 Github上的独立项目。Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。（摘抄自百度百科）

​	也就是说，Netty是一个网络应用程序框架，是用来开发和网络服务有关的，所以本次的TCP服务端就使用的是Netty。同时，Netty是基于Java中的NIO的，但是Netty对NIO的封装比较友好。（大家都说java中的NIO烂的一塌糊涂，我也没有用过，只能顺大流了）另外，Netty吸收了很多的协议的实现经验，所以保证了易于开发并且性能稳定。说完了它的历史背景和所解决的问题，下面就要开始介绍一下Netty的一些基础组件，方便后面的理解。

* 引导类。引导类分为服务端ServerBootStap、客户端BootStrap。引导类作用是将各个基础组件通过链式语法进行组装，服务端通过bind()进行启动,客户端通过connect()进行启动。

* 线程模型。
  * Boss线程：负责接收客户端连接请求。
  * Worker线程：负责IO读写事件和任务处理(比如channel注册selector、channel绑定端口等)，通过inEventLoop()和MPSC(Multiple Producer Single Consumer)队列实现无锁化串行线程执行模型。
  * IO-Reactor：Netty可配置三种IO-Reactor线程模型,分别为单Boss线程处理客户端连接和IO读写、单Boss线程处理客户端连接-多Worker线程处理IO读写、多Boss线程处理客户端连接-多Worker线程处理IO读写。
  * NioEventLoopGroup：线程池，内部存储可用线程NioEventLoop，默认大小为2倍的cpu核数。
  * ThreadPerTaskExecutor：线程创建器，创建过程中会为线程重命名，并将其转换为FastThreadLocalThread类型。
  * Chooser：线程选择器，根据NioEventLoopGroup中线程数量采取不同策略选取可用线程。
  
* 通道/业务逻辑处理
  * Channel：数据传输通道,Netty对NIO中ServerSocketChannel和SocketChannel进行抽象和功能封装。
  * ChannelPipeline：业务逻辑处理链，采取责任链模式将事件进行传递交由ChannelHandler进行处理。Netty通过Synchronized关键字保证ChannelPipeline的线程安全性，实现ChannelHandler的动态添加和删除。事件传递：入站事件向后一个入站事件传递，出站事件向前一个出站事件传递，异常事件向后一个事件传递。
  * ChannelHandler：业务处理组件，分为入站处理、出站处理和入出站处理三种类型。特别注意ChannelHandler并非线程安全，因此最好实现一些无状态方法，或者通过线程安全容器和关键字保证数据的线程安全。
  * ChannelHandlerContext：存储业务处理组件上下文信息，保存ChannelHandlerContext->ChannelHandler的1:1映射关系，因为ChannelHandler可被共享重复使用，所以ChannelHandler->ChannelHandlerContext并不一定是一一映射。
  
* 数据容器/编解码

  * ByteBuf：Netty中数据存储的载体，根据内存类型、内存回收和API实现机制可分为Head/Direct、Pooled/UnPooled和UnSafe/非UnSafe三种类型。编解码：数据以二进制方式在服务端和客户端进行传输，接收数据后需要对数据进行解码，发送数据前需要对数据进行编码。
  * ByteToMessageDecoder：解码器，将累加的字节流数据进行decode，decode操作交由子类实现，通过fireChannelRead事件将解码后的数据向后传播。
  * MessageToByteEncoder：编码器，分配内存将数据写入ByteBuf，发起write事件将编码后的数据向前传播。

​	上面就是基于作用来介绍各个组件，还有一个图，这些都是拷贝自另一个博主的，总结的挺好的，但是不知道为什么不写了。。。博客地址：[https://www.jianshu.com/p/17ccfc1005f8](https://www.jianshu.com/p/17ccfc1005f8)

![image-20200618155425968](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200618155425968.png)

  下面就说一下Netty的特点。（来自官网）

* 丰富的缓冲区数据结构
* 通用的异步I/O API
* 基于拦截器链模式的事件模型
* 高级组件，实现更快的开发

还有一个官网的图，看不懂也要看。。。

![image-20200618153050073](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200618153050073.png)

​	感觉还是官网比较靠谱，我在各个博客找了很久，大部分都不是很好，后来感觉官网写挺不错的，同时官网上面也有jar包和文档，都比较齐全。

​	官网地址：[https://netty.io](https://netty.io)

​	后面就是开始写demo了，简单介绍到这里就结束了，后面有机会也慢慢补上其他的概念。

​	就这样吧，结束。

