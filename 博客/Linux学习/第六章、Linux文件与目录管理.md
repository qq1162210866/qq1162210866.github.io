# Linux学习
## 第六章、Linux文件与目录管理
### 文件与目录管理	
* `ls`命令

  ![IMG_E867F1DE7BD3-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_E867F1DE7BD3-1.jpeg)

* 复制、删除与移动  `cp`,`rm`,`mv`
![IMG_83380C6DF2B8-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_83380C6DF2B8-1.jpeg)
### 文件内容查阅		
* `cat`：从第一行显示到最后一行	
![IMG_76BB5886D069-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_76BB5886D069-1.jpeg)
* `tac`：从最后一行显示到第一行		
* `nl`：显示的时候显示行号	
![IMG_14F24DA98032-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_14F24DA98032-1.jpeg)
* `more`：一页一页的显示文本	
![IMG_F97B32F53055-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_F97B32F53055-1.jpeg)
* `less`：一页一页显示文本，但是可以往前翻页	
![IMG_777F6AABB74F-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_777F6AABB74F-1.jpeg)
* `head`：只看头几行	
![IMG_B955711F9456-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_B955711F9456-1.jpeg)
* `tail`：只看后面几行		
  ![IMG_9D3335EA23B0-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_9D3335EA23B0-1.jpeg)
* `od`：以二进制的形式读取文件内容	
![IMG_1F0EC4C06432-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_1F0EC4C06432-1.jpeg)
取一个文件11行到20行到内容		
`head -n 20 /test.txt | tail -n 10	`	
管线符号`｜`：前面的指令所输出的信息，通过管线命令交给后续指令继续使用	
显示行号：`cat -n test.txt | head -n 20 | tail -n 10`		
* `touch`修改文件时间或创建一个新的文档
![IMG_F1CC616035BD-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_F1CC616035BD-1.jpeg)			

###  文件与目录的默认权限和隐藏权限		
`umask`指定用户在创建文件或者目录时的默认权限值
![IMG_00741A334223-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_00741A334223-1.jpeg)
默认权限值代表的意思
![IMG_39339A93FE77-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_39339A93FE77-1.jpeg)

* `chattr`配置文件隐藏的属性（和下面的一个命令一样不是很常用）
  ![IMG_DDB37D20B3DD-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_DDB37D20B3DD-1.jpeg)

* `lsattr`显示文件的隐藏属性

  ![IMG_8CF328C04A07-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_8CF328C04A07-1.jpeg)

* 文件的特殊权限：SUID,SGID,SBIT

  SUID指文件的执行者在执行程序时拥有文件拥有者相同的权限，一般出现在文件拥有者的x权限上。

  ![IMG_27D05CA751BD-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_27D05CA751BD-1.jpeg)

  SGID是SUID的升级版，升级为文件的群组。

  ![IMG_8541875233A9-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_8541875233A9-1.jpeg)

  SBIT只针对目录，在该目录下创建的文件只有自己或者root才能修改或者删除

  ![IMG_9CF18B040559-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_9CF18B040559-1.jpeg)

  授权上面的三种特殊权限可以通过数字或者字符

  ![image-20200412204346040](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200412204346040.png)

![image-20200412204449320](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200412204449320.png)

* `file`命令用来查看文件的类型
* ![IMG_31B8AC9CC1EB-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_31B8AC9CC1EB-1.jpeg)

### 指令与文件的搜寻

* `which`寻找可执行的文档

  ![IMG_2760EE6A96EE-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_2760EE6A96EE-1.jpeg)

* `whereis`寻找系统中某些特定目录底下的文件(用处估计不是很多，但是速度快)

  ![IMG_00AD66018E29-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_00AD66018E29-1.jpeg)

* `locate`命令是在系统建立的数据库（一天修改一次的数据库）中查询文件。一般配合`updatedb`使用（用到的地方也是很少）

  ![IMG_1169BF8BB661-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_1169BF8BB661-1.jpeg)

![IMG_4F1C1F7A6DD6-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_4F1C1F7A6DD6-1.jpeg)

* `find`命令是一个非常强大的命令的，但是是因为直接读取的硬盘，所以速度没有上面两个快。

  ![IMG_7FC983BD2FC3-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_7FC983BD2FC3-1.jpeg)

![IMG_D52E7DA4D5A9-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_D52E7DA4D5A9-1.jpeg)

![IMG_FD693BD4905F-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_FD693BD4905F-1.jpeg)

![IMG_3EDE5760B28D-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_3EDE5760B28D-1.jpeg)

![IMG_A65852973ABA-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_A65852973ABA-1.jpeg)

自己的理解，和管线命令类似。但是是以参数的形式![image-20200412214544680](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200412214544680.png)