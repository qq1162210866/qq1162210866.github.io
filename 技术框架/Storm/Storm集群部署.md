## Storm集群部署

​	上一篇博客写了一个demo，在IDEA中也跑成功了，下面就是部署到集群环境中去运行，也是搞了一天，和普通的Java项目有点不太一样。开始。

​	部署Storm集群比较繁琐，网上资料也鱼龙混杂，所以还是建议大家去官网上找官方文档。

​	Storm官方部署流程：[http://storm.apache.org/releases/2.2.0/Setting-up-a-Storm-cluster.html](http://storm.apache.org/releases/2.2.0/Setting-up-a-Storm-cluster.html)

​	首先就是下载基础的东西，jdk和Strom、zookeeper安装包。地址如下：

​	JDK下载地址：[https://www.oracle.com/java/technologies/javase-downloads.html](https://www.oracle.com/java/technologies/javase-downloads.html)

​	Storm下载地址：[http://storm.apache.org/downloads.html](http://storm.apache.org/downloads.html)

​	ZooKeeper下载地址：[https://zookeeper.apache.org/releases.html](https://zookeeper.apache.org/releases.html)

​	需要注意的是下载的时候注意，Storm和ZooKeeper需要下载发行版本，不要下载源码版本，要不然会出现问题，很蛋疼。如下图

​	![image-20200710104725796](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200710104725796.png)

​	下载完毕，放到自己指定的目录，解压就可以了。一步一步来吧。

* 安装JDK

  1.查看是否已经安装jdk

  rpm -qa|grep jdk

  2.卸载原有jdk，因为openjdk不适用

  yum -y remove java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64

  3.将jdk安装包上传到服务器上。将其解压到指定位置。记得先创建文件夹

  tar -zxvf jdk-8u191-linux-x64.tar.gz -C /usr/local/java/

  4.在/etc/profile文件末尾添加

  export JAVA_HOME=/usr/local/java/jdk1.8.0_191

  export JRE_HOME=${JAVA_HOME}/jre

  export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib

  export PATH=${JAVA_HOME}/bin:$PATH

  5.使/etc/profile文件生效

  source /etc/profile

  6.验证安装是否成功

  Java -version

* 安装ZooKeeper

  Zookeeper有一个官方安装文档，很详细。地址：[https://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_systemReq](https://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_systemReq)

  需要注意的就是，Storm使用Zookeeper协调集群。ZooKeeper 不用于消息传递，因此Storm在Zookeeper上的负载非常低。在大多数情况下，单节点Zookeeper集群就足够了。所以我安装的就是单节点的ZooKeeper。按照文档来就可以了，没有什么可以讲的点，如果不行在网上搜一下错误信息。
  
* 安装Storm

  把Storm的包解压到指定目录，修改配置文件，这一步需要注意的是，例如：`storm.local.dir: "/mnt/storm"`前面是有一个空格的，要不然启动时会抱错误。安装流程文档就是最上面的一个链接。

  修改完毕配置文件后，依次启动服务。`bin/storm nimbus`,`bin/storm nimbus`,`bin/storm ui`。因为UI服务默认占用的是8080端口，如果有修改需要，只需要在配置文件添加：`ui.port: 8083`（注意空格）。

  剩下就没有什么了，启动三个服务后，输入一下网址：`http://{ui host}:8080`就可以看到Storm的UI，如下图：

  ![image-20200710110638215](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200710110638215.png)

  这里就不再介绍各个组件的意思了，都很容易看懂。

* 提交自己的Topology

  提交自己的Topology也很简单，官网上也有详细的介绍，地址：[http://storm.apache.org/releases/2.2.0/Running-topologies-on-a-production-cluster.html](http://storm.apache.org/releases/2.2.0/Running-topologies-on-a-production-cluster.html)

  需要注意的是，如果你的Storm没有加入环境变量，需要使用全路径，如下：

  `/apache-storm-2.2.0/bin/storm jar strom-0.0.1-SNAPSHOT.jar com.example.strom.StromApplication count ` 

  另外，打包的时候也需要注意，要指定主类，要不然会一直报找不到主类的错误。

  打包需要注意去除SpringBoot的依赖，因为他本身带有打包插件，打包会报错误。

  打包插件代码如下：

  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-shade-plugin</artifactId>
              <executions>
                  <execution>
                      <phase>package</phase>
                      <goals>
                          <goal>shade</goal>
                      </goals>
                      <configuration>
                          <transformers>
                              <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                  <mainClass>com.example.strom.StromApplication</mainClass>
                              </transformer>
                          </transformers>
                      </configuration>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
  ```
  
  上传正常截图如下：
  ![image-20200710114634347](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200710114634347.png)
  删除Topology截图如下：
  ![image-20200710112836552](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200710112836552.png)
  
  StormUI截图：
  
  ![image-20200710114916495](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200710114916495.png)
  

​	剩下就没有什么要说的了。这样就初步完成集群部署了。

​	就这样吧，结束。