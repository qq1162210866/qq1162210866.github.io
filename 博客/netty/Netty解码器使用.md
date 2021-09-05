## Netty解码器了解和使用

​	距离上次了解Netty已经过去半年了，最近需要编写一个客户端去连接一个设备，发现Netty编写客户端比服务端复杂一点，回头看自己写的服务端的博客，发现当时的了解也是一知半解，整片博客达不到我想象中的要求。现在才发现写博客原来还是有一些技术含量的。我也没有修改写完的博客的习惯。就当成自己的黑历史吧，但是后续自己还是要记录的。中间发现一片博客对于Netty的入门的一些概念讲的比较清楚，这里也放上来。[Netty架构与原理](https://mp.weixin.qq.com/s/insjE_EJRoCOM-1GqgZP9A)

​	也就不多说废话了，这边博客讲的是解码器的使用，关于客户端的讲解和代码放到后面一片博客上。下面就直接开始。

​	编码器和解码器其实是一个概念，上一篇博客讲过Netty中的几个重要的组件，其中编解码器在最下面。解码器意如其名，就是对于网络上接收的字节进行解码，将字节转换为相应的java对象，再传给业务处理类，同时对于TCP中的粘包和拆包现象在解码器中处理也是比较方便的。关于粘包问题的解释和做法可以看官网的解释[官网对于粘包的解释和处理](https://netty.io/wiki/user-guide-for-4.x.html#dealing-with-a-stream-based-transport)。最开始我的本意也是处理粘包的现象，同时对收到的数据做校验。所以就查询了一下相关的资料。编码器则是发送数据时将java对象转换为字节数据，再通过网络发送出去，和解码器类似，只不过用的地方不一样。我的代码不涉及发送数据，所以本次博客不再记录编码器，估计使用方法和解码器类似，可能就是方法不一样。

​	Netty想到了粘包和拆包的问题，所以有几种自带的解码器，可以直接使用。LineBasedFrameDecoder（基于特殊分隔符的解码器，例如`\r\n`）、DelimiterBasedFrameDecoder（更加通用的分隔符解码器，支持多个特殊分隔符）、FixedLengthFrameDecoder（固定长度的解码器）、LengthFieldBasedFrameDecoder（可以按照长度字段动态的解码器，比较符合我们的要求了）。基本常用的解码器就这几种了，最后一个比较符合我们的需求，但是因为后续可能会发生变化，同时，我也想在解码器里面对数据做校验，所以这里打算自己编写解码器。

​	自定义解码器只需要让类继承ByteToMessageDecoder，然后重写自己的方法就可以了。这里我记得Netty有几种解码器，分别解析int short和二进制，但是忘记到底是什么了，欢迎大家补充。

​	继承ByteToMessageDecoder后，只需要重写`decode()`方法就可以完成解码的工作，这里简单列举一下自己的代码，因为大部分都涉及到业务处理，这部分和解码器无关，不再记录。代码如下：

```java
/**
     * Description: 先处理粘包的问题，再对所有数据和数据模块做校验
     *
     * @param ctx ChannelHandler 和ChannelPipeline 之间的关联
     * @param in 输入的Buf
     * @param out 输出的Buf
     * @return void
     * @Author: Peng Shiquan
     * @Date: 2020/11/30
     */
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
      	//获取buf中写指针的位置，相当于buf中能读的数据的长度
        int bufferLength = in.writerIndex();
      	//获取指定帧头的位置
        int headIndex = ByteBufUtil.indexOf(RadarMessageUtil.HEAD_BUF, in);
        int surplusLength = RadarMessageUtil.HEAD_LENGTH + RadarMessageUtil.SUM_LENGTH + RadarMessageUtil.CALIBRATE_LENGTH;
      	//获取当前数据最短的长度，帧头+数据长度位+数据+校验位
        int shortData = in.getUnsignedShort(headIndex + RadarMessageUtil.HEAD_LENGTH) + surplusLength;
        if (headIndex >= 0 && bufferLength >= shortData) {
            int dataLength = shortData - surplusLength;
            byte[] rs = new byte[dataLength];
            in.getBytes(headIndex + RadarMessageUtil.HEAD_LENGTH + RadarMessageUtil.SUM_LENGTH, rs);
            if (compareBytes(rs)) {
              	//添加该buf到输出中，这里也可以将buf再次转换为java对象
                out.add(rs);
            }
          	//将读写指针复位
            in.readerIndex(headIndex + RadarMessageUtil.HEAD_LENGTH + RadarMessageUtil.SUM_LENGTH + dataLength + RadarMessageUtil.CALIBRATE_LENGTH);
            in.discardReadBytes();
        } else {
            logger.info("接收的数据不满足最小长度或者不包含帧头");
        }
    }
```

​	代码就是上面的代码，不是很复杂。简单说一下思路。先是获取整个缓冲区的数据长度，再判断帧头的位置，将帧头后两位的数据解析为数据长度，判断缓存区整个数据长度是不是大于这个长度，如果大于就代表这个包全了，可以解析，将这个buf转换成字节数组再放到out里面（不是必须转换为字节数组）。如果小于则代表缓冲区内数据不全，需要继续等待缓冲区内数据补充，直接返回就可以。如果正常读取后，最后需要将buf中的指针操作都复位，要不然buf的大小会越来越小。

​	以上就是一个解决特定帧头，动态长度的解码器。当然，这个解码器还可以做的更多，例如对数据做校验，这里也就不再展开了。运行效果和网上其他的类似，能够解决相应的粘包问题。使用也比较简单，在添加业务处理类前添加解码器即可。代码如下：

```java
 Bootstrap b = new Bootstrap();
        RadarConnectThread radarConnectThread = this;
        b.group(eventLoopGroup).channel(NioSocketChannel.class)
                .option(ChannelOption.TCP_NODELAY, true)
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 6000)
                .option(ChannelOption.SO_KEEPALIVE, false)
                .handler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel socketChannel) {
                        socketChannel.pipeline().addLast(new RadarDecoder()).addLast(new RadarHandler(radarConnectThread));
                    }
                });

        return b;
```

​	中间还是有一些点讲的不是很清楚，后续还会补充一片Netty客户端的博客，这篇博客更加全面的记录，同时修正之前博客的错误（又是一个坑）。

​	那就这样吧，结束。

