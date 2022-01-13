---
title: MySQL的连接方式
date: 2022-01-13 20:22:57
tags: MySQL
---
## MySQL的连接方式

​	今后会慢慢写一些关于MySQL的博客，补上自己这一部分的知识。今天这篇博客就先介绍一下MySQL连接的方式，主要还是因为连接MySQL时发现了一个参数。

`-Djdk.tls.disabledAlgorithms=SSLv3, TLSv1, RC4, DES, MD5withRSA, DH keySize < 1024, EC keySize < 224, 3DES_EDE_CBC, anon, NULL, include jdk.disabled.namedCurves`

### Linux的登录方式

​	要谈MySQL的连接方式，我想先说一下Linux的认证方式。登录有两种方式：本地登录和远程登录，本地登录直接命令行或者页面输入密码即可，没有可讲的点。远程登录又有四种方法。SSH、Telnet、VNC和SFTP 4种。

* SSH：SSH(Secure Shell)是一种使用加密技术保护传输数据包的远程登录工具，所有数据包都先经过加密，再进行传输。由于SSH是一种安全性非常高的远程登录工具，因此SSH也是Linux系统中使用最广泛的远程登录方式。平时我们常用的就是这种。

* Telnet：Telnet是一个传统的交互式登录工具。与SSH不同的是，Telnet并没有使用加密技术，所有内容都通过明文方式传输。由于其保密性差，因此通常将其应用到能够确认安全的环境下，例如一些私有网络等。我平时使用这个命令一般都是测试端口是否通畅。

* VNC：VNC(Virtual Network Computing，虚拟远程计算机)是由AT&T欧洲实验室开发的一个用于远程控制的开源软件，在Linux系统中主要用于远程桌面控制。远程软件，还是挺出名的。

* SFTP：SFTP(Secure File Transfer Protocol，安全文件传输协议)是SSH的一部分，主要用来在Linux系统间传送文件。这个用的少。

​	既然说完了Linux的远程连接方式，下面就顺带说一下我比较迷糊的点。

### SSH、SSL、TLS的区别

​	SSH刚才说过了，是一种远程登录加密的工具，主要用于shell方面，原理比较复杂，这里我才疏学浅，放上别人的博客当作一个参考，本文也不再详细探讨。[SSH原理](https://www.jianshu.com/p/d31de2601368)

​	SSL和TLS都是安全协议，SSL同时也是TLS的前身。https中的那个s就是指的SSL，最开始SSL诞生就是为了互联网浏览器的加密，后来慢慢标准化为TLS，慢慢成为广大应用程序的安全协议。

​	TLS1.0和SSL3.0几乎没有区别，事实上我们现在用的都是TLS，但因为历史上习惯了SSL这个称呼，平常还是以SSL为多。

​	SSL(Secure Sockets Layer 安全套接层),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层对网络连接进行加密。

​	SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。SSL协议可分为两层： SSL记录协议（SSL Record Protocol）：它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。 SSL握手协议（SSL Handshake Protocol）：它建立在SSL记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。

​	他们两个的区分也可以参考维基百科的解释：[维基百科](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)

​	回到文章开头的代码，这段代码其实是一个参数，但不是MySQL的参数，是Java运行的参数，含义就是禁止使用这些安全协议：`SSLv3, TLSv1, RC4, DES, MD5withRSA`。

### MySQL的连接方式

​	终于讲到了MySQL连接方式了。。。

​	截止到MySQL5.7为止，总共有四种连接方式，分别是TCP/IP，Unix Sockets，Shared Memory，Named pipes。需要注意的是，这些连接方式所支持的系统是不一样的，最后两个连接方式只支持Windows系统，Unix Sockets只支持类Unix系统，例如Centos、Mac OS。TCP/IP因为是网络连接，所以所有的系统都支持。我这里因为暂时没有Windows电脑，所以举例也是只使用前两种。

#### Unix Sockets

​	这个其实比较常见，例如我在一台Centos电脑上安装MySQL，安装完毕后，直接在这台电脑上执行以下命令`mysql -uroot -p12345678`,便会出现以下画面。（因为我是Docker容器内运行的，可能部分细节不一致，但是大体是差不多的）

![image-20220113200026860](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20220113200026860.png)

​	这种方式比较普遍，没什么可以说的点，注意不要命令行中输入你的密码。我这里因为MySQL配置问题，只能明文输入密码。

#### TCP/IP

​	这个就是我们大家通常的连接方式了，安装一个MySQL连接客户端，指定IP地址连接。输入密码就可以连接了。命令如下：

​	`mysql -P3306 -h127.0.0.1 -uroot -p12345678 --ssl=on`

​	执行后的截图如下：

![image-20220113200619524](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20220113200619524.png)

​	剩下的几种方式也是大同小异，这里就不再举例了。

### 结束语

​	这里也参考了一些博客和文档，有官方的，也有一些前辈的，这里一块放上来，供大家参考。

​	[官方文档](https://www.docs4dev.com/docs/zh/mysql/5.7/reference/encrypted-connections.html)

​	[对于MySQL连接的解释](https://zhuanlan.zhihu.com/p/43941022)

​	就这样吧，结束。

