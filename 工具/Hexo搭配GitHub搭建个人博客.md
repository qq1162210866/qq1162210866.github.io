

https://sspai.com/post/59337

使用hexo

https://zhuanlan.zhihu.com/p/26625249



ssh-keygen -t rsa -b 4096 -C "1162210866@qq.com"

 模板用A-Ayer

博客主分支使用静态文件部署

博客的source分支使用md文档构成

平时更新md文档

然后编译打包到主分支





# 使用Hexo和GitHub搭建个人博客

本篇博客认为读者已经有一些编程基础，很多开发使用的工具和一些简单的命令，这里不再赘述。并且需要注意，本篇博客使用的主机为centos7.6系统搭建，其他操作系统或者其他版本可能部分命令不能使用。

## 前置条件

​	搭建之前先说一下前置条件。其实就是申请厂库，因为GitHub能够将你账号下的一个特殊厂库里面的内容转换为可以显示的web页面，所以你需要创建指定名称的厂库，格式为：`<user>.github.io`,注意，user设置为你的GitHub账号名。

* 申请GitHubPage，申请你的仓库即可，注意你厂库的名称。
* 再电脑上生成ssh文件为了远程连接厂库。

```bash
ssh-keygen -t rsa -b 4096 -C "你的邮箱"
cat /root/.ssh/id_rsa.pub
```

将全部内容复制到github的页面上，记得可以带上`ssh-rsa`。可以设置这个密钥只能访问这个仓库。

## 安装步骤

### 安装nodejs16

这里简单的放上步骤，这里的安装就是安装nodejs官方的yum源，然后通过yum命令安装，大家如果感兴趣也可以看看官方的脚本是如何编写的。

```bash
curl -fsSL https://rpm.nodesource.com/setup_16.x | bash -
sudo yum install -y nodejs
```

测试nodejs安装完毕后

### 安装hexo

​	这个安装就很简单了，使用npm安装软件即可。

执行安装hexo即可

```bash
npm install hexo-cli -g
HEXO -V
```

### 初始化

到执行目录，执行hexo init

目录结构

https://hexo.io/zh-cn/docs/setup

### 对hexo进行设置

​	项目初始化完毕后，就可以修改你hexo的一些配置了。

配置仓库的信息

### 编译和发布

​	编译使用命令

​	发布需要注意git需要安装，然后设置自己的git邮箱

安装git和hexogit插件

设置git用户名称，和你的ssh邮箱一致

hexo g

生成静态文件

hexo d

部署文件

完成

### 配置插件和主题

剩下就是设置你自己的博客主题了

开始愉快的编写博客吧



```
---
title: hello hexo markdown
date: 2016-11-16 18:11:25
tags:
---
```





