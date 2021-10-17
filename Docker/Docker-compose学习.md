## Docker-compose学习

​	前两个月因为公司的事情，没有时间写写博客（其实是自己给自己找借口）。后面慢慢来，补上自己欠下的技术帐。这篇博客就是之前定的计划，但是没有机会写。下面就直接开始。

​	先说说Docker Compose的作用，有一部分人分不清楚Dockerfile和Docker Compose的区别，Dockerfile是用来构建镜像的构建文件，相当于告诉Docker如何构建一个自己需要的镜像。这个文件后面也会有一个博客来单独介绍。Docker Compose是用来编排容器，如果你要启动多个容器的话，使用单个命令多次执行是很繁琐的，并且每个容器需要的参数也可能不一致。Docker Compose就是解决这个问题的，使用统一编排启动多个容器，并且对多个容器进行不同的设置。下面就来简单介绍大体的结构，单个容器不同的参数配置这里就不在叙述了，可能就是简单举个例子。

​	首先需要先安装Docker Compose，算是Docker的一个插件，安装也很简单，mac和linux上面的安装我都安装过，但是Windows上的没有，所以找了官方的安装文档，上面各个系统都有，虽然是英文的，但是使用翻译还是能够看懂的，作为程序员，我比较后悔大学没有学好英语。[Docker Compose官方安装文档](https://docs.docker.com/compose/install/)

​	安装完毕，就需要了解如何使用和如何编写。一步一步来。

​	编写前，需要先了解Docker Compose和Docker版本的匹配图，要不然无法编写合适的Docker Compose文件。如下图：

![image-20201117143036674](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20201117143036674.png)

​	下面就一个简单的例子，来列举Docker Compose中对于一个容器的配置。

```yml
version: "3.8"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
networks:
  frontend:
  backend:

volumes:
  db-data:
```

​	例子有了，现在就介绍一下这个例子中的各个参数的含义，由上往下开始说吧。`version`是指这个Docker Compose文件的版本，上面一个图也是指定不同Docker版本和Docker Compose的对应关系，这里是官网的例子，所以用的是3.8.`services`指的是该yml文件需要启动哪些服务，上面的例子是启动redis服务，redis服务中指定了镜像的来源、容器的端口、容器网络的配置、部署时的配置。在下面则是`networks`，声明了两个网络，名称分别是`fronted`和`backend`，最后则是一个卷的声明，声明了一个`db-data`，需要注意的是，这个卷需要在docker-compose启动前建立，要不然会报错，这个后面会讲。以上就是对于这个文件的一个初步讲解。可以看到这个文件对于docker里面的很多知识点都联系了起来，所以想要随心所欲的写出这样的一个文件，对于文件进行任意的修改是需要长时间docker的使用的，这里本人的技术也是有限，所以暂时讲到这里，对于一些细节的补充和知识点的错误，欢迎大家的指正。

​	下面就开始列举一个我自己编写的一个文件，并且讲一下其中遇到的问题和难点。文件内容如下：

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

  mysql:
    image: mysql:5.7
    labels:
      service: mysql
    ports:
      - 3306:3306
    container_name: mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=12345678
      - TZ=Asia/Shanghai
    volumes:
      - mysqldata:/var/lib/mysql/
    logging:
      driver: 'json-file'
      options:
        labels: "service"
        max-size: '30m'
        max-file: '1'

  redis:
    image: redis:3.2.9
    labels:
      service: redis
    container_name: "redis-server"
    environment:
      - TZ=Asia/Shanghai
    ports:
      - "6379:6379"
    volumes:
      #- ./data:/data:z
      - ../conf/redis.conf:/usr/local/etc/redis/redis.conf
    restart: always
    command: [ "redis-server", "/usr/local/etc/redis/redis.conf" ]
    logging:
      driver: 'json-file'
      options:
        labels: "service"
        max-size: '30m'
        max-file: '1'
        
volumes:
  elasticdata:
    driver: local
    driver_opts:
      o: bind
      type: volume
      device: /home/program/data/elasticdata
  mysqldata:
    driver: local
    driver_opts:
      o: bind
      type: volume
      device: /home/program/data/mysqldata

networks:
  elastic:
    driver: bridge
```

​	这里是列举了三个容器的配置文件，这里其实有很多不对的地方，但是目前能力和精力有限，也没有来得及修复。三个容器使用了卷，分别为`elasticdata`和`mysqldata`卷，网络只有ES使用了桥连接，但是也没有做好相应的设置。redis将本地的配置文件放进容器中，用来做密码和用户的设置，但是Mysql直接将密码当作环境变量，在容器启动的时候输入。上面说了，Docker Compose牵扯到许多Docker的知识点，想要随心所欲的编写文件很不容易，这里的配置许多是看了网上的配置和官方推荐的配置。自己理解的话很费时间也比较困难，只能鼓励自己慢慢爬上这座大山，也是留个坑，后面遇到这类问题也会慢慢的补充。本次的记录就到此为止了。

​	就这样吧，结束。

