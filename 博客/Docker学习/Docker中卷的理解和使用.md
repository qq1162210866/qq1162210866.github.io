## Docker中卷的理解和使用

​	最近工作的需要，了解了一下Docker的相关内容，同时接触了Docker中volume（下面都以卷来称呼）的概念，这篇博客也来记录一下自己学习到的东西。

​	Docker中的文件都是存储在容器层，这样做的好处是将宿主机和Docker内的容器隔离开来，但是缺点也是很明显的，当容器停止后，Docker不会存储容器运行中创建的文件，所以容器一旦停止，容器产生数据都会丢失，于是Docker提供了两个方法来将容器的文件存储到宿主机中。*volumes*, and *bind mounts*.绑定很好理解，就是声明将容器内的某个文件或者某个文件夹绑定到宿主机的某个文件或者文件夹，只需要在Docker-compose文件中声明即可（我的做法）。但是这种方法并不是Docker官方推荐的做法，官方更加推荐通过卷的形式来管理数据，理由也有以下几点。

- 与绑定安装相比，卷更易于备份或迁移。
- 您可以使用Docker CLI命令或Docker API管理卷。
- 卷在Linux和Windows容器上均可工作。
- 可以在多个容器之间更安全地共享卷。
- 卷驱动程序使您可以将卷存储在远程主机或云提供程序上，以加密卷内容或添加其他功能。
- 可以通过容器预填充新卷的内容。

  所以本次也来介绍卷的理解和使用。**卷**存储在*由Docker管理*的主机文件系统的一部分中（`/var/lib/docker/volumes/`在Linux上）。非Docker进程不应该去修改这个目录下的文件。卷是在Docker中持久保存数据的最佳方法。创建卷时，它存储在Docker主机上的目录中。将卷装入容器时，此目录就是装入容器的目录。这类似于绑定挂载的工作方式，卷由Docker管理并且与主机的核心功能隔离。说了这么多，下面就来开始使用吧。

	### 卷的创建

​	卷的创建比较简单，一条命令即可。`docker volume create test`.查看Docker中的卷也是比较简单,`docker volume ls`.查看特定卷的详细信息：`docker volume inspect test`.下面列出执行结果。

![image-20200920175606271](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200920175606271.png)

### 卷的使用

​	卷最终还是要容器里使用的，下面就以一个nginx的镜像来使用刚刚创建的test卷。

```bash
docker run -d --name=nginx -v test:/usr/share/nginx/html nginx:latest
```

​	这里需要注意，如果-v指定的卷没有创建，Docker会自动创建的。但是这个不是本次要介绍的方法，因为Docker-compose比较好用，所以工作中整合也是使用Docker-compose来自动创建卷，并且将卷挂载到哦宿主机的某个目录。Docker-compose文件内容如下：

```yaml
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
    container_name: elasticsearch
    restart: always
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - cluster.initial_master_nodes=es01
      - http.port=9200
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elasticdata:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - elastic
 volumes:
  elasticdata:
    driver: local
    driver_opts:
      o: bind
      type: volume
      device: /home/program/data/elasticdata  
```

​	因为对于Docker-compose和卷了解的还不是很清楚，所以这一部分可能会使用错误或者理解错误。这个文件会启动时会自动创建卷和将宿主机目录挂载到卷上。也欢迎大家到指正。

![image-20200920181837154](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200920181837154.png)

​	这里疑惑的点有两个，一个是卷的名称和我文件中指定的不一致，还有就是上面图片显示的两个目录都存储的有ES的数据。因为自己的能力有限，目前无法理解这两个地方，后续填坑吧。

​	上面的知识点都是来自于官网，官网地址如下：[Docker中数据的管理](https://docs.docker.com/storage/)，[Docker中volume命令详解](https://docs.docker.com/engine/reference/commandline/volume/)。

​	就这样吧，结束。