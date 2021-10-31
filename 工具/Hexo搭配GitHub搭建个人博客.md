# 使用Hexo和GitHub搭建个人博客

​	本篇博客认为读者已经有一些编程基础，很多开发使用的工具和一些简单的命令，这里不再赘述。并且需要注意，本篇博客使用的主机为centos7.6系统搭建，其他操作系统或者其他版本可能部分命令不能使用。

## 前置条件

​	搭建之前先说一下前置条件。其实就是申请git仓库，因为GitHub能够将你账号下的一个特殊仓库里面的内容转换为可以显示的web页面，所以你需要创建指定名称的仓库，格式为：`<username>.github.io`,注意，user设置为你的GitHub账号名。

* 申请GitHubPage，申请你的仓库即可，注意你仓库的名称。
* 在电脑上生成ssh文件为了远程连接仓库。

```bash
ssh-keygen -t rsa -b 4096 -C "你的邮箱"
cat /root/.ssh/id_rsa.pub
```

​	将全部内容复制到github的页面上，记得可以带上`ssh-rsa`。可以设置这个密钥只能访问这个仓库。

![github设置仓库密钥](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/github设置仓库密钥.PNG)

​	现在你就可以尝试放一些页面来看看效果了，访问路径为：`https://<username>.github.io`

​	下面就开始安装Hexo了。

## 安装步骤

​	在安装Hexo需要使用npm，所以需要安装Node.js，低版本的Node.js会因为兼容问题导致所无法使用Hexo，所以这里直接安装版本16.

### 安装Node.js16

​	这里放上安装步骤，这里的安装就是安装nodejs官方的yum源，然后通过yum命令安装，大家如果感兴趣也可以看看官方的脚本是如何编写的。

```bash
curl -fsSL https://rpm.nodesource.com/setup_16.x | bash -
sudo yum install -y nodejs
```

​	测试nodejs安装完毕后。

![安装nodejs](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/安装nodejs.PNG)

### 安装hexo

​	这个安装就很简单了，使用npm安装软件即可。

```bash
npm install hexo-cli -g
HEXO -V
```

![安装hexo](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/安装hexo.PNG)

### 初始化

​	可以创建一个博客目录，执行`hexo init`,会自动在当前目录下创建对应的Hexo各个子目录。各个目录的作用可以看下面的链接。[官方文档对于目录的解释](https://hexo.io/zh-cn/docs/setup)

### 对Hexo进行配置

​	项目初始化完毕后，就可以修改你Hexo的一些基础配置了。例如提交到你仓库的设置。

​	配置仓库的信息：

![hexo部署配置](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/hexo部署配置.PNG)

​	还有一些配置是个性化的配置，这个看自己喜好。关于如何修改这些配置项，官方也有相关的文档。地址如下：

[官方配置文件解读](https://hexo.io/zh-cn/docs/configuration)。

### 编译和发布

​	如果你简单写了一个文档，想要先看看效果，就可以执行编译和发布了。这里我没有本地建站，所以是直接推送到GitHub仓库。

​	编译就是将你的md文档转换为对应的html文档。发布就是将你的html文件推送到指定的仓库或者本地建立一个包含你html文件的web服务。命令很简单，就是`hexo g`和`hexo d`。分别对应编译和发布。

​	如果你需要发布到GitHub，需要注意你要先安装一个插件，就是一个git插件，让Hexo能够调用git上传代码。

​	编译成功就是如下的截图：

![部署完成](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/部署完成.PNG)

​	以上就是一个简单的Hexo搭配GitHub的使用了。	

### 配置插件和主题

​	我想大部分人自己搭建博客就是为了个性化，要不然市面上主流的博客网站就够用了。所以Hexo也支持自定义主题。大家可以在官网上搜索自己喜欢的主题，然后按照主题的要求和步骤完成切换。这里因为各个主题的安装步骤都不一致，所以我也就不再举例子了。我这里就简单上一个官方主题商店吧，大家可以看看有没有自己中意的。[官方主题商店](https://hexo.io/themes/)

​	然后就是插件了，因为Hexo不能完全照顾到所有人，所以就有一些开发者自己开发了插件，方便大家的使用。例如我使用的就有一款插件，能够根据你文件夹的名称自动生成分类，并且将你的博客进行归档，还是很方便的。虽然没能完全满足我的需求，但也是挺不错的了。这里也是放上插件和官方插件商城的链接。[官方插件商城地址](https://hexo.io/plugins/)，[自动生成类别插件](https://github.com/xu-song/hexo-auto-category)

​	因为时间和精力的原因，我的博客折腾之旅就结束了，虽然还是有一些小瑕疵，但是基本能满足我的使用。后续如果又有兴趣了，我就再接着折腾吧。

​	就这样吧，结束。

