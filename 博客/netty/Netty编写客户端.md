## Netty编写客户端

​	上一篇博客讲了一下解码器，但是其实没有涉及到客户端的编写，今天补上这篇博客。同时深入了解一下Netty（对于我来说）。加深自己的印象。

![image-20201208205126779](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20201208205126779.png)

​	上面是一个简单的服务端的例子，之前的博客也讲过这个demo，下面按照自己最新的理解再次记录一下。首先是最开始创建两个线程组，boss线程组用来监听客户端的连接，只做到这一步，work线程组相当于干活的，当通道中有IO事件时就开始工作。这样做的好处就是可以提升读写性能。这一块的内容涉及到Netty的线程模型，这一块我不是很熟悉，找到一篇文章，大家可以了解一下，感兴趣的可以继续搜索一下：[Netty线程模型](https://zhuanlan.zhihu.com/p/87630368)，以后也会慢慢补上这些内容的，学习无涯。回到代码上，然后就是创建了一个`ServerBootstrap`，这个启动类的作用就是辅助与编写服务端，降低开发的复杂度。再然后就是设置线程组、设置`Channel`、配置TCP的参数，编写Netty之前最好还是有点NIO的编写经验，不要像我上来就开始，一点准备工作都没有做，读代码都很吃力。最后的两行代码含义就比较容易懂了，调用辅助类的`bind()`方法绑定监听某个端口，同时调用同步阻塞方法等待绑定操作完成。最后一行则是等待服务端链路关闭后，退出主方法。

​	说了这么多，其实和客户端的编写一点关系都没有，主要还是记录一下Netty启动的流程，对于Netty原理和底层的一些操作，这个需要等我看完NIO再记录一下。下面就讲一下Netty编写客户端的思路。

​	客户端不需要两个线程组，一个就够了，通道也和服务端不太一样，需要设置为`NioSocketChannel`,剩下就是设置TCP的参数和handler的设置。发起连接和服务端也不太一样，服务端是监听，客户端就是连接，调用`connect()`方法发起异步连接，然后再调用同步方法等待连接成功。最后就是连接关闭后释放线程组的资源。代码也比较简单，如下：

```java
//配置客户端的线程组，客户端只有一个线程组
EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
try {
    Bootstrap bootstrap = new Bootstrap();
    bootstrap.group(eventLoopGroup).channel(NioSocketChannel.class)
            .option(ChannelOption.TCP_NODELAY, true)
            .handler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel socketChannel) throws Exception {
                    //放入自己的业务Handler
                    socketChannel.pipeline().addLast(new TCPClientHandler());
                }
            });
    //发起异步连接操作，同步阻等待结果
    ChannelFuture channelFuture = bootstrap.connect(host, port).sync();
    //等待客户端链路关闭
    channelFuture.channel().closeFuture().sync();
} finally {
    //释放NIO线程组
    eventLoopGroup.shutdownGracefully();
}
```

​	不要以为到这里就结束了，上面这个只是一个简单的demo，自己写着玩还可以，但是到了生产环境或者实际用的时候，很多东西没有考虑到。断线重连问题、线程管理的问题、连接多服务端的情况。这些问题是我开发时遇到的问题，下面就一一讲一下实现的逻辑，最后则给出整体的代码。

* 断线重连。断线重连一般时客户端要做的事情，也分为两种情况，一开始连接不上和连接上过一段时间断了。这两种情况需要不同处理，一开始连接不上需要添加监听器，异步执行，失败就调用两秒重连。断线则需要再handler中的`channelInactive()`方法做相应的业务处理。
* 线程管理和连接多服务端的问题其实是一个场景。就是需要连接多个服务端的情况下，一个连接创建一个线程显然是不太合理的，同时也要对这些线程进行管理。这里暂时的办法是单独启动一个线程，线程的`run()`方法就是启动netty连接。后续在这个线程里面去完成多个客户端的建立或者一个netty线程去连接多个服务端。目前水平有限，只能想到这个地步。涉及到线程和NIO还有Netty。后续慢慢完善吧。



​	所以最后的代码就如下：因为一些原因，没有给出全部的代码，给出了部分核心的代码，其他的细节和本次的内容关系不大。也可以按照自己的需求进行修改。（最好按照自己的想法修改一下，每个人的需求都不太一样，我的代码一定不合适所有人）

```java
public class ConnectThread extends ConnectThread {
    private Logger logger = LoggerFactory.getLogger(ConnectThread.class);

    private Bootstrap bootstrap;

    private EventLoopGroup eventLoopGroup;

    private AtomicBoolean running = new AtomicBoolean(true);

    @Override
    public void run() {
        beginConnect();
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                shutDown();
            }
        });
        //阻塞住
        while (running.get()) {
            try {
                Thread.sleep(200l);
            } catch (Exception e) {
                logger.error("程序运行发生错误");
                e.printStackTrace();
                shutDown();
            }
        }
    }

    public void beginConnect() {
        try {
            eventLoopGroup = new NioEventLoopGroup();
            bootstrap = new Bootstrap();
            assembleBootStrap();
            reConnect();
        } catch (Exception e) {
            logger.error("客户端连接发送错误");
            e.printStackTrace();
            shutDown();
        }
    }

    /**
     * Description: 组装BootStrap
     *
     * @param
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/12/9
     */
    public void assembleBootStrap() {
        ConnectThread ConnectThread = this;
        bootstrap.group(eventLoopGroup).channel(NioSocketChannel.class);
        bootstrap.remoteAddress(host, port);
        bootstrap.option(ChannelOption.TCP_NODELAY, true);
        bootstrap.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 6000);
        bootstrap.option(ChannelOption.SO_KEEPALIVE, true);
        bootstrap.handler(new ChannelInitializer<SocketChannel>() {
            @Override
            protected void initChannel(SocketChannel socketChannel) {
                socketChannel.pipeline().addFirst(new IdleStateHandler(10, 0, 0))
                        .addLast(new Decoder())
                        .addLast(new Handler(ConnectThread));
            }
        });
    }

    @Override
    public void reConnect() {
        super.reConnect();
        logger.info("雷达开始重试，雷达信息为：{}:{}", host, port);
        bootstrap.connect(host, port).addListener(new ConnectListener(this));
    }

    public void shutDown() {
        running.set(false);
        if (closeChannel != null && closeChannel.isActive()) {
            closeChannel.close().awaitUninterruptibly();
            closeChannel = null;
        }
        bootstrap = null;
        if (eventLoopGroup != null) {
            eventLoopGroup.shutdownGracefully();
        }
        eventLoopGroup = null;
        logger.error("客户端关闭连接，雷达服务端信息为：{} : {}", host, port);
    }
}
```

​	来讲一下上面的代码大致的思路。连接类继承了线程，重写了`run()`方法，run方法里面只有三个部分的代码，一个是启动连接，一个是程序运行中有异常执行关闭方法，释放资源。最后一个就是将线程阻塞住，因为启动连接是异步的，如果不阻塞，代码直接就执行完毕了，后续也没法进行重连的操作。启动连接方法里面只有两步，一个是组装BootStrap启动类，另外一个就是执行重连操作，之前说过，断线重连发生在两个地方，最开始没有连接上和中间断掉，所以这里直接掉同一个方法，不用再重复写方法了。组装启动类按照自己的需求组装就可以了，设置超时事件、设置长时间无数据返回、设置解码器，业务处理类。重连方法就是调用启动类的`connect()`方法，这里和上面的demo有一点不一样，这里是异步返回的，没有阻塞住，添加了一个监听器，也是为了做重连用的。如果和demo一样的话，这里发生断连，程序就直接关闭了，就没法重连了。想要重连必须重新创建线程组、启动类、通道等，这个显然不是我们想看到了，因为断连后，线程组和和启动类还是可以继续使用的，只不过要换一个连接。

​	下面就继续附上监听器和handler的代码。（handler中有一部分断线重连的方法）

```java
public class ConnectListener implements ChannelFutureListener {
    private Logger logger = LoggerFactory.getLogger(ConnectListener.class);

    private final ConnectThread ConnectThread;

    public ConnectListener(ConnectThread ConnectThread) {
        this.ConnectThread = ConnectThread;
    }

    @Override
    public void operationComplete(ChannelFuture future) throws Exception {
        if (future.isSuccess()) {
            logger.info("[" + ConnectThread.getHost() + ":" + ConnectThread.getPort() + "] 启动成功!!!");
            ConnectThread.setTime(0);
            ConnectThread.setCloseChannel(future.channel());
        } else {
            if (ConnectThread.getTime() >= 1800) {
                logger.error(ConnectThread.getHost() + ":" + ConnectThread.getPort() + "重试超过一小时，不再重试");
                return;
            }
            logger.warn("[" + ConnectThread.getHost() + ":" + ConnectThread.getPort() + "]Channel失联，2秒后重连。。。 ");
            future.channel().eventLoop().schedule(new Runnable() {
                @Override
                public void run() {
                    ConnectThread.reConnect();
                }
            }, 2l, TimeUnit.SECONDS);

        }
    }
}
```

​	监听器的代码就是判断是否连接上，没有的话进行重试，连接上了就重置次数，同时返回通道信息，为后面关闭通道做准备。剩下句没有什么可以说的了。最好就是handler的代码。

```java
public class XYDHandler extends ChannelInboundHandlerAdapter {
    private Logger logger = LoggerFactory.getLogger(XYDHandler.class);

    private XYDConnectThread xydConnectThread;

    public XYDHandler(XYDConnectThread xydConnectThread) {
        this.xydConnectThread = xydConnectThread;
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        logger.error("连接断开，IP端口为：{}", ctx.channel().remoteAddress());
        ctx.channel().eventLoop().schedule(() -> {
            xydConnectThread.reConnect();
        }, 2, TimeUnit.SECONDS);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf byteBuf = Unpooled.copiedBuffer((byte[]) msg);
        logger.info("接收到消息为：{}", ByteBufUtil.hexDump(byteBuf));
    }
```

​	重连的方法就是`channelInactive()`，和监听器里的代码也是差不多。剩下就没有什么可以说的了。

​	上面的代码就是完成一个断线重连的Netty的客户端，代码直接粘贴是不能用的，需要进行一些修改，我现在也比较烦直接将代码粘贴过来，上来就开始测试。还是要了解一下具体如何实现的，要不然改代码都不知道如何修改。就像这次编写客户端，最开始写很吃力，因为之前是一知半解，不知道如何修改，不知道如何排查，所以走了很多的弯路，代码完全不能看，也不起用。现在了解了一些后，慢慢能够按照自己的想法将代码改造成自己想要的样子，我觉得这样才算一个东西你彻底掌握了，与君共勉。

​	就这样吧，结束。