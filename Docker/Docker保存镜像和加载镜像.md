## Docker保存镜像和加载镜像

​	最近在搞一键部署，因为Docker的镜像下载很慢，所以想搞成离线加载的，这样可以省去很多时间，于是了解了一下Docker的镜像保存和加载机制，下面就直接开始吧。

​	这里是直接将你本地的镜像打包，这样就可以在另一台机器上加载这个镜像了。下面就列出来用到的几条命令，同时实际操作一下。

*  查询要保存的镜像

  ​	用到的命令：`docker ps`，需要用到的就是`REPOSITORY`或者`IMAGE ID`,这个是镜像存放的仓库，相当于云上的地址，后面也会有博客讲一下如何将自己的镜像推到云上，这里不在耽误时间了。直接上图。

  ![image-20200919174552480](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200919174552480.png)

* 保存镜像

  ​	用到的命令`docker save [OPTIONS] IMAGE [IMAGE...]`,参数只有一个-o,表示输出的文件名。这里直接打包命令为：

  `docker save -o test.tar 121454ddad72` 或者

  `docker save -o test1.tar docker.elastic.co/elasticsearch/elasticsearch`

  ![image-20200919175202935](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200919175202935.png)

* 加载镜像

  这个就是最后一步了，加载也是比较简单的，把刚才的tar包上传到服务器，直接load即可。命令如下:

  `docker load -i test.tar`

  加载后到镜像和打包的镜像是一样的，版本和仓库地址。后续就可以愉快使用了。如图：

  ![image-20200919175508640](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200919175508640.png)




  这次的博客比较短，也没有什么可说的，后续慢慢补充吧。

  就这样吧，结束。