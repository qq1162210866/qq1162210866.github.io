## 第七章、Linux文件系统
### Linux文件系统

* Linux的EXT2文件系统

  ![IMG_3DFC7CA9B512-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_3DFC7CA9B512-1.jpeg)

  * Data block 是用来放置文件内容数据的地方。Ext2系统中所支持的blokc大小有1、2，4K三种。

    ![IMG_2488E99AAFB4-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_2488E99AAFB4-1.jpeg)

  * Inode table 是记录文件的属性以及该文件实际数据是放在哪个block中的。

    ![IMG_F2D95A3DEA87-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_F2D95A3DEA87-1.jpeg)
    
    inode还有下面的特点。
    
    ![IMG_7173E6AAB4C5-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_7173E6AAB4C5-1.jpeg)
    
    inode的记录结构图，最小的1K的block可以容纳16GB的文件。`12+256+256*256+256*256*256=16GB`
    
    ![IMG_B0821E59ADEE-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_B0821E59ADEE-1.jpeg)
    
   * Superblock记录整个filesystem相关信息的地方。

     ![IMG_163F8E9E2A81-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_163F8E9E2A81-1.jpeg)

   * Filesytem Description（文件系统描述说明）描述每个block group的开始与结束的block号码，以及说明每个区段分别介于哪一个block号码之间。
  
   * Block bitmap(区块对照表)记录系统中空的block号码。
  
   * Inode bitmap（inode 对照表）记录未使用的inode号码。
  
  * `dumpe2fs` 查询Ext家族superblock信息的指令。（不常用）
  
    ![IMG_51A8DF0CE1DB-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_51A8DF0CE1DB-1.jpeg)
    
  * 文件的读取流程
  
    ![IMG_7F443C85D318-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_7F443C85D318-1.jpeg)

* XFS文件系统简介

  * 资料区。分为多个存储群组，每个群组包含（1）整个文件系统的 superblock、 (2)剩余空间的管理机制、 (3)inode 的分配与追踪。

  * 文件系统活动登录区。主要记录文件系统的变化，有点类似于日志区。

  * 实时运作区。记录extent区块的信息。

  * `xfs_info`指令的用法

    ![IMG_368E80FB9615-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_368E80FB9615-1.jpeg)

* 文件系统的简单操作

  * `df`列出文件系统的整体磁盘使用量

    ![IMG_0AAC648A473B-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_0AAC648A473B-1.jpeg)

   * `du`评估文件系统的磁盘使用量
  
     ![IMG_01964E14EB78-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_01964E14EB78-1.jpeg)
  
* 实体链接与符号链接

  * `ln`制作链接档。

    ![IMG_C5F5EAF2A276-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_C5F5EAF2A276-1.jpeg)

    * Hard Link（实体链接，硬式连结或实际连结）在某个目录下新增一笔档名链接到某inode号码的关联记录。

    * Symbolic Link(符号链接，类似于快捷方式) 建立一个独立的文件，而这个文件会让数据的读取指向link的那个文件档名。会创建新的inode和block，但是要比原档名小的多。

* 磁盘的分区、格式化、检验与挂载

  * `lsblk`列出系统上所有的磁盘列表。
    ![IMG_881CF83F6037-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_881CF83F6037-1.jpeg)
    
   * `blkid`列出装置的UUID等参数
  
   * `parted`列出磁盘的分区表类型与分区信息
  
   * 分区的步骤：先通过`lsblk`或者`blkid`找到磁盘，再通过`parted /dev/xxx print` 找到该磁盘的分区类型，之后通过`fdisk`(MBR分区)或者`gdisk`(GPT分区)来操作磁盘。
  
   * `mkfs.xfs`命令是建置文件系统
  
     ![IMG_C576B170E89D-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_C576B170E89D-1.jpeg)
  
   * EXT4文件系统的`mkfs.ext4`
  
     ![IMG_D34A4CD5C7F0-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_D34A4CD5C7F0-1.jpeg)
  
* 文件系统的检验

  * `xfs_repair`处理XFS文件系统
    ![IMG_ED943AA7B08E-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_ED943AA7B08E-1.jpeg)

  * `fsck.ext4`处理EXT4文件系统

    ![IMG_1AC76C5CAAB8-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_1AC76C5CAAB8-1.jpeg)

* 文件系统的挂载和卸除

  * `mount`挂载使用到的命令

    ![IMG_EF0C6D310CCA-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_EF0C6D310CCA-1.jpeg)

  * `umount`将装置文件卸除的命令

    ![IMG_D4FFFBEBEEE1-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_D4FFFBEBEEE1-1.jpeg)
  
* 设置开机挂载

  * /etc/fstab文件内容

    ![image-20200421091505802](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200421091505802.png)

    ![IMG_DA7EB7846934-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_DA7EB7846934-1.jpeg)

    * 磁盘装置文件名/UUID/LABEL name。 例子：/dev/vda2.  UUID=xxx.  LABEL=xxx

    * 挂载点。一定要是目录

    * 磁盘分区槽的文件系统。如xfs,ext4,vfat,reiserfs,nfs

    * 文件系统参数

      ![IMG_01E1BA15FC1D-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_01E1BA15FC1D-1.jpeg)

    * 能否被dump备份指令作用。0即可

    * 是否以fsck检验扇区。xfs文件系统会自己检查，直接填0即可

 * 特殊装置loop挂载

    * 以挂载一个文件为例

      ![image-20200421134709795](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200421134709795.png)

![image-20200421134803389](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20200421134803389.png)

步骤：1.建立大型文件。2.大型文件格式化。3.将大型文件挂载到目录（通过UUID的形式）

* 内存置换空间（swap）之建置（用处不大，看看就可以）

  ![IMG_57EEF927240A-1](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/IMG_57EEF927240A-1.jpeg)

  