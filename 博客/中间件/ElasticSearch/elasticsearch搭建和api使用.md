# Elasticsearch搭建和简单API使用

​	之前的博客介绍了SpringBoot如何使用ES，但是ES最新版本的安装没有说明，最近需要做这个了发现没有具体的文档，这篇博客就来说一下ES在CentOS下的安装文档，同时也介绍一下一些简单的ES使用的API。不多逼逼了，开始。
### Elasticsearch安装

* 下载RPM包。

  本次的安装步骤是使用RPM包安装的，因为步骤比较简单。所以第一步需要下载RPM包，这个需要到官网下载，地址如下：[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)。根据自己的需要下载对应的版本就可以了。把RPM包上传到服务器上即可。

* 安装ES

  在RPM包目录下执行命令：

  ~~~bash
  rpm  --install elasticsearch-7.9.1-x86_64.rpm
  ~~~

  出现以下画面就表示安装完成

  ![image-20200904104228179](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200904104228179.png)

* 配置ES

  配置ES前需要先安装Java，但是这个比较简单，并且不在本次叙述内容里面，这里就不再累述了，网上博客一搜一大堆。

  * 配置ES中的JAVA_HOME。

    配置一下ES中的JAVA_HOME，这样ES启动的时候就会自己去找Java环境了。

    ~~~bash
    vim /etc/sysconfig/elasticsearch
    
    修改JAVA_HOME属性为本机的配置即可
    ~~~

  * 修改ES配置文件

    ~~~bash
    vim /etc/elasticsearch/elasticsearch.yml
    
    修改以下配置
    cluster.initial_master_nodes
    network.host
    http.port
    node.name
    cluster.name
    详细文件内容如下：
    # ======================== Elasticsearch Configuration =========================
    #
    # NOTE: Elasticsearch comes with reasonable defaults for most settings.
    #       Before you set out to tweak and tune the configuration, make sure you
    #       understand what are you trying to accomplish and the consequences.
    #
    # The primary way of configuring a node is via this file. This template lists
    # the most important settings you may want to configure for a production cluster.
    #
    # Please consult the documentation for further information on configuration options:
    # https://www.elastic.co/guide/en/elasticsearch/reference/index.html
    #
    #
    # ---------------------------------- Cluster -----------------------------------
    #
    # Use a descriptive name for your cluster:
    #
    cluster.name: elasticsearch
    #
    # ------------------------------------ Node ------------------------------------
    #
    # Use a descriptive name for the node:
    #
    node.name: node-1
    #
    # Add custom attributes to the node:
    #
    #node.attr.rack: r1
    #
    # ----------------------------------- Paths ------------------------------------
    #
    # Path to directory where to store the data (separate multiple locations by comma):
    #
    path.data: /var/lib/elasticsearch
    #
    # Path to log files:
    #
    path.logs: /var/log/elasticsearch
    #
    # ----------------------------------- Memory -----------------------------------
    #
    # Lock the memory on startup:
    #
    #bootstrap.memory_lock: true
    #
    # Make sure that the heap size is set to about half the memory available
    # on the system and that the owner of the process is allowed to use this
    # limit.
    #
    # Elasticsearch performs poorly when the system is swapping the memory.
    #
    # ---------------------------------- Network -----------------------------------
    #
    # Set the bind address to a specific IP (IPv4 or IPv6):
    #
    network.host: 0.0.0.0
    #
    # Set a custom port for HTTP:
    #
    http.port: 9200
    #
    # For more information, consult the network module documentation.
    #
    # --------------------------------- Discovery ----------------------------------
    #
    # Pass an initial list of hosts to perform discovery when this node is started:
    # The default list of hosts is ["127.0.0.1", "[::1]"]
    #
    #discovery.seed_hosts: ["host1", "host2"]
    #
    # Bootstrap the cluster using an initial set of master-eligible nodes:
    #
    cluster.initial_master_nodes: ["node-1"]
    #
    # For more information, consult the discovery and cluster formation module documentation.
    #
    # ---------------------------------- Gateway -----------------------------------
    #
    # Block initial recovery after a full cluster restart until N nodes are started:
    #
    #gateway.recover_after_nodes: 3
    #
    # For more information, consult the gateway module documentation.
    #
    # ---------------------------------- Various -----------------------------------
    #
    # Require explicit names when deleting indices:
    #
    #action.destructive_requires_name: true
    ~~~

* 启动ES，添加到开机自启

  命令如下：

  ~~~bash
  service elasticsearch start
  chkconfig --add elasticsearch
  ~~~

* 其他信息

  * 删除ES

    ~~~bash
    rpm -e elasticsearch-7.9.1-x86_64.rpm
    ~~~

  * 开放防火墙端口（建议开启防火墙）

    ~~~bash
    firewall-cmd --zone=public --add-port=9200/tcp --permanent
    ~~~

  * 把日志目录到权限赋予es用户
  
    ~~~bash
    chown -R es:es /opt/elasticsearch
    ~~~

### API简单介绍

​	ES可以通过使用RESTful API来进行交互，使用的是9200端口。更极端的情况下，可以使用curl命令来进行交互。

​	一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：

~~~bash
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
~~~

|      参数      |                             介绍                             |
| :------------: | :----------------------------------------------------------: |
|     `VERB`     | 适当的 HTTP *方法* 或 *谓词* : `GET`、 `POST`、 `PUT`、 `HEAD` 或者 `DELETE`。 |
|   `PROTOCOL`   | `http` 或者 `https`（如果你在 Elasticsearch 前面有一个 `https` 代理） |
|     `HOST`     | Elasticsearch 集群中任意节点的主机名，或者用 `localhost` 代表本地机器上的节点。 |
|     `PORT`     |    运行 Elasticsearch HTTP 服务的端口号，默认是 `9200` 。    |
|     `PATH`     | API 的终端路径（例如 `_count` 将返回集群中文档数量）。Path 可能包含多个组件，例如：`_cluster/stats` 和 `_nodes/stats/jvm` 。 |
| `QUERY_STRING` | 任意可选的查询字符串参数 (例如 `?pretty` 将格式化地输出 JSON 返回值，使其更容易阅读) |
|     `BODY`     |          一个 JSON 格式的请求体 (如果请求需要的话)           |

* 例子

  * 创建一个商品的索引，指定四个属性

    ~~~bash
    http://localhost:9200/commodity
    
    {
        "settings":{
            "number_of_shards":3,
            "number_of_replicas":2
        },
        "mappings":{
            "properties":{
                "commodity_id":{
                    "type":"long"
                },
                "commodity_name":{
                    "type":"text"
                },
                "picture_url":{
                    "type":"keyword"
                },
                "price":{
                    "type":"double"
                }
            }
        }
    }
    
    返回结果：
    {
        "acknowledged": true,
        "shards_acknowledged": true,
        "index": "commodity"
    }
    ~~~

    ![image-20200904143546873](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200904143546873.png)

  还有一些用法，这里就不再列出来了，详细的文档可以看看官方文档，官方文档是基于2.0版本的，但是安装的是7.9，所以部分用法可能有些不一样，但是了解ES的一些基础的东西估计没有问题。中文文档地址在下方。[https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)



​	这样就基础的就差不多了，就这样吧，结束。