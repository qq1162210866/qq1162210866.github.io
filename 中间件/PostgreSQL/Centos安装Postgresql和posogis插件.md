## Centos安装Postgresql和posogis插件

​	公司临时让装一个数据库，正好是之前了解过的Postgresql，顺带把之前的安装的坑填。并且这次也安装了插件。下面直接开始。

​	安装的插件有：fuzzystrmatch、postgis、uuid-ossp。其中uuid-ossp需要再初始化数据库的时候指定，剩下的都可以在安装完数据库后进行安装。下面直接上教程，不逼逼了。很详细。

* 下载postgresql官方压缩包，选择对应的版本。地址：[https://www.postgresql.org/ftp/source/](https://www.postgresql.org/ftp/source/)，下载tar.gz

![image-20200806103650919](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806103650919.png)

* 解压到指定目录

  ~~~bash
   tar -xvf postgresql-9.6.10.tar -C /home/program/soft/
  ~~~

* 执行配置命令，中间出现部分错误，按照指示安装对应插件即可。

  ~~~bash
  yum install uuid uuid-devel
  ./configure --prefix=/bigdata/work/postgresql --with-uuid=ossp
  ~~~

  ![image-20200806104151703](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806104151703.png)

  ~~~bash
   yum install readline-devel
  ~~~

  ![image-20200806104250201](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806104250201.png)

  ~~~bash
  yum install zlib-devel
  ~~~

  出现下面命令即可以执行make命令。

  ![image-20200806104818429](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806104818429.png)

* 执行make命令

  ~~~bash
  make
  make install
  ~~~

  出现以下截图即安装成功：

  ![image-20200806105752457](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806105752457.png)

* 后续的配置

  创建一个postgres用户，创建数据目录和日志目录，将权限给这个用户。

  ~~~bash
  adduser postgres
  mkdir /bigdata/work/postgresql/data
  chown -R postgres:postgres /bigdata/work/postgresql/data
  ~~~
  
* 添加环境变量
  
  ~~~bash
  vim /etc/profile
  
  export PGDATA=/bigdata/work/postgresql/data
  export PGHOME=/bigdata/work/postgresql
  export PATH=$PGHOME/bin:$PATH
  
  source /etc/profile
  ~~~
  
* 初始化数据库
  
  ~~~bash
  su - postgres
  initdb
  ~~~
  
  ![image-20200806111651152](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806111651152.png)
  
* 修改两个配置，使用root用户

  ~~~bash
  vim /bigdata/work/postgresql/data/pg_hba.conf
  vim /bigdata/work/postgresql/data/postgresql.conf
  ~~~

  ![image-20200806141658615](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806141658615.png)

  ![image-20200806141724073](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806141724073.png)

* 将启动脚本复制到linux中去管理。

  从postgresql中的解压目录复制

  ~~~bash
  cp contrib/start-scripts/linux  /etc/init.d/postgresql
  vim /etc/init.d/postgresql
  ~~~

  ![image-20200807090320846](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200807090320846.png)

* 授予权限，设置开机自启

  ~~~bash
   chmod +x /etc/init.d/postgresql
   chkconfig --add postgresql
  ~~~

* 启动，设置密码。

  直接使用root用户启动

  ~~~bash
  service postgresql start
  ~~~

  使用postgrees用户查看进程信息

  ![image-20200806142450969](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806142450969.png)

  使用自带的工具连接数据库，设置密码。可以讲这个设置为系统环境变量。

  ~~~bash
  /bigdata/work/postgresql/bin/psql
  \password
  ~~~

* 使用远程工具连接测试

  ![image-20200806142714037](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200806142714037.png)

  

* 安装Postgis插件

  * 先做准备工作，设置postgres用户的环境变量

  ~~~bash
  su - postgres
  vim ~/.bash_profile
  
  export PGDATA=/bigdata/work/postgresql/data
  export PGHOME=/bigdata/work/postgresql
  export PATH=$PGHOME/bin:$PATH
  export LD_LIBRARY_PATH=$PGHOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/lib:/usr/lib:/usr/local/lib:$LD_LIBRARY_PATH
  ~~~

  * 安装需要依赖包

  ~~~bash
   yum install -y python-devel perl-ExtUtils-Embed python-devel gcc-c++ openssl-devel readline readline-devel bzip2 zlib zlib-devel openssl openssl-devel pam pam-devel libxml2 libxml2-devel libxslt libxslt-devel openldap openldap-devel libgeos-dev libproj-dev libgdal-dev xsltproc docbook-xsl docbook-xml imagemagick libmagickcore-dev dblatex tcl tcl-devel unixODBC unixODBC-devel libpng12 libpng12-devel
  ~~~

  * 下载其他依赖包

    * 安装Proj4
  
      ~~~bash
    wget http://download.osgeo.org/proj/proj-4.9.3.tar.gz
      tar -xf proj-4.9.3.tar.gz
      cd proj-4.9.3
      ./configure --prefix=/bigdata/work/postgresql/plugin/proj
      make 
      make install
      echo "/bigdata/work/postgresql/plugin/proj/lib" > /etc/ld.so.conf.d/proj-4.9.3.conf
      ldconfig
      ~~~
    
    * 安装GEOS
    
      ~~~bash
      wget http://download.osgeo.org/geos/geos-3.6.1.tar.bz2
      tar -jxf geos-3.6.1.tar.bz2
      cd geos-3.6.1
    serivice jj
      make
      make install
      echo "/bigdata/work/postgresql/plugin/geos/lib" > /etc/ld.so.conf.d/geos-3.6.1.conf
      ldconfig
      ~~~
    
    * 安装GDAL
    
      ~~~bash
      wget http://download.osgeo.org/gdal/2.1.2/gdal-2.1.2.tar.gz
      tar -xf gdal-2.1.2.tar.gz
      cd gdal-2.1.2
      ./configure --prefix=/bigdata/work/postgresql/plugin/gdal
      make
      make install
      echo "/bigdata/work/postgresql/plugin/gdal/lib" > /etc/ld.so.conf.d/gdal-2.1.2.conf
      ldconfig
      ~~~
    
    * 安装PostGIS
    
      ~~~bash
      wget http://download.osgeo.org/postgis/source/postgis-2.2.5.tar.gz
      tar -xf postgis-2.2.5.tar.gz
      cd postgis-2.2.5
      ./configure --prefix=/bigdata/work/postgresql/plugin/postgis --with-pgconfig=/bigdata/work/postgresql/bin/pg_config --with-geosconfig=/bigdata/work/postgresql/plugin/geos/bin/geos-config --with-gdalconfig=/bigdata/work/postgresql/plugin/gdal/bin/gdal-config --with-projdir=/bigdata/work/postgresql/plugin/proj
      make
      make install
      ~~~
    
      ![image-20200807094928599](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200807094928599.png)
    
      解决这个问题。需要将postgresql的lib目录导入ld.so.conf.d
    
      ~~~bash
      vim /etc/ld.so.conf
      
      /bigdata/work/postgresql/lib
      
      ldconfig
      ~~~
  
  * 验证插件正常安装
  
    ~~~bash
    su postgres
    psql
    create database postgis;
    \c postgis
    CREATE EXTENSION postgis;
    CREATE EXTENSION postgis_topology;
    CREATE EXTENSION fuzzystrmatch;
    CREATE EXTENSION postgis_tiger_geocoder;
    CREATE EXTENSION "uuid-ossp";
    ~~~

    
  
  ![image-20200807150426313](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200807150426313.png)
  
  即成功。
  
  fuzzystrmatch是在源码中的有的，但是没有包含编译的时候没有包含进去，需要make一下。
  
    ~~~bash
    cd contrib/fuzzystrmatch/
    make
    make install
    ~~~
  
  UUID-OSSP插件也是
  
    ~~~bash
     cd contrib/uuid-ossp/
     make
     make install
    ~~~
  
    如下图：
  
    ![image-20200807151502098](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200807151502098.png)
  



​	剩下就没有什么可以讲的了，按照步骤来就可以了。以为一些centos的依赖指定了版本，如果你的PostgreSQL版本和我的不一致，这一点需要注意。如何查询对应的版本需要到网上搜一搜。

​	就这样吧，结束。