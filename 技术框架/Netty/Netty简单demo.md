## Netty简单demo

​	上篇博客简单介绍了Netty的一些基础概念和组件，这次就来写一个简单的demo，因为公司需要的是TCP服务端，所以这次的demo就写一个TCP服务端。下面就直接上代码。

​	首先是主类，里面也放着Netty的启动方法和执行流程。代码如下：

```java
package nettytrain;


import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

/**
 * TCPServerTrain.java
 * Description:  netty服务端的练习
 *
 * @author Peng Shiquan
 * @date 2020/6/18
 */
public class TCPServerTrain {
    //默认端口
    private Integer port = 10000;

    public TCPServerTrain(Integer port) {
        this.port = port;
    }

    public void run() throws Exception {
        // 接收传入的连接
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        //处理传入的连接，一般是bossGroup的二倍
        EventLoopGroup workGroup = new NioEventLoopGroup();
        try {
            //服务端应用开发的入口
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            //设置主从线程池
            serverBootstrap.group(bossGroup, workGroup)
                    //指定通道channel的类型，因为是服务端，所以是NioServerSocketChannel
                    .channel(NioServerSocketChannel.class)
                    //设置子通道也就是SocketChannel的处理器
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new TCPServerHandler());
                        }
                    })
                    //配置ServerSocketChannel的选项
                    .option(ChannelOption.SO_BACKLOG, 128)
                    //配置子通道也就是SocketChannel的选项
                    .childOption(ChannelOption.SO_KEEPALIVE, true);
            //绑定并侦听某个端口
            ChannelFuture channelFuture = serverBootstrap.bind(port).sync();
            //如何没有客户端连接就会关闭Channel和两个线程池
            channelFuture.channel().closeFuture().sync();
        } finally {
            /**
             * 关闭线程池
             */
            workGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }

    }

    public static void main(String[] args) throws Exception {
        int port = 10000;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        }
        new TCPServerTrain(port).run();

    }

}
```

​	代码里面的注释已经很清楚了，没有什么可以细讲的地方。下面的就是业务处理类，自己的业务处理主要放在这里。代码：

```java
package nettytrain;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;
import io.netty.util.ReferenceCountUtil;
import network_train.ARPojo;
import network_train.ARSwitch;

/**
 * TCPServerHandler.java
 * Description: 主要的业务处理类
 *
 * @author Peng Shiquan
 * @date 2020/6/18
 */
public class TCPServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * Description: 通道的读取操作方法
     *
     * @param ctx
     * @param msg
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/6/19
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        try {
            ByteBuf recvmg = (ByteBuf) msg;
            String line = recvmg.toString(CharsetUtil.UTF_8);
            System.err.println("接收到消息" + line);
            if (line.contains("BG") && line.contains("ED")) {
                ARPojo arPojo = ARSwitch.LineSwitch(line);
                System.err.println("转换为pojo对象" + arPojo.toString());
            }
        } finally {
            // 释放msg
            ReferenceCountUtil.release(msg);
        }
    }


    /**
     * Description:  异常捕获
     *
     * @param ctx
     * @param cause
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/6/19
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

​	也是没有什么可以讲的地方，这里的业务处理我放在其他的地方了，所以这里的代码很少，每个人的业务不同，所以业务处理也不同。

​	下面就说说遇到的问题。就是通道读取的方法，刚开始不知道怎么处理msg，后来转换成`ByteBuf`,再转换的`String`，这个地方不知道处理的对不对。其他的问题就是这个代码没有考虑到粘包的现象，现在还没有涉及到，所以没有写，后面慢慢补上。

​	就这样吧，结束。