​	又是工作中遇到了一个Maven的问题，解决了好久都是迷迷糊糊的，这次记录下来防止自己再次忘记。

## Maven是什么

​	Maven 是一个意第绪语单词，意思是知识的积累，最初是为了简化 Jakarta Turbine 项目中的构建过程。每个项目都有自己的 Ant 构建文件，都略有不同。我们想要一种构建项目的标准方法、对项目组成的清晰定义、一种发布项目信息的简单方法以及一种在多个项目之间共享 JAR 的方法。

​	结果是现在可以用于构建和管理任何基于 Java 的项目的工具。我们希望我们已经创建了一些东西，使 Java 开发人员的日常工作更容易，并且通常有助于理解任何基于 Java 的项目。

​	这个是Maven官网上对于Maven是什么的一个介绍。也可以看出Maven这个工具的用处，就是简单的管理项目依赖的jar包，但是随着Maven的发展，后续也增加了许多有用的功能。想我们使用的话，也就是简单的管理jar包或者依赖。

## 配置Maven环境

​	 配置Maven环境不是本篇博客的重点，所以不在这里详细的介绍，只是说一下大致的环节。

​	 安装Java环境，将Java的安装地址配置到系统的环境变量里面。下载Maven压缩包，解压到你指定的目录，配置Maven的系统环境变量即可。最后修改一下settings文件，修改镜像地址和本地仓库地址即可。

​	执行`mvn -version`出现以下画面就代表安装成功。

![image-20210902214851086](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902214851086.png)

## 简单使用Maven

​	Maven安装完毕，下面就可以搞一个简单的小Demo，看到成功的画面才有学下去的动力，这个Demo也是官方的例子，具体的地址如下： [Maven简单Demo](https://Maven.apache.org/guides/getting-started/Maven-in-five-minutes.html)

​	在目录下执行如下命令：

`mvn archetype:generate -DgroupId=com.psq.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false`

![image-20210902215259888](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902215259888.png)

​	这个命令就是依据你选定的原型创建了一个Maven结构的项目，项目名称为`my-app`。项目的结构如下：

![image-20210902215525898](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902215525898.png)

​	可以看到和我们平时的SpringBoot应用相差不多，其实SpringBoot应用也是按照Maven的原型修改了一下。

​	下面执行`mvn package`，可以看到

![image-20210902215723521](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902215723521.png)

​	这里因为我依赖都下载过了，所以很快，第一次执行会稍微慢一些。能够看到项目目录中多了一些东西，多出来的东西就是Maven编译和打包产生的文件，还有一些其他的文件，目前暂时不清楚。

![image-20210902215910039](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902215910039.png)

​	我们再执行命令`java -cp ./target/my-app-1.0-SNAPSHOT.jar com.psq.app.App `就能够看到一个输出了。到此，Maven的简单Demo就结束了。

![image-20210902220047445](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902220047445.png)

​	这个例子完成是不是对于Maven的使用更加的清晰了。下面就介绍一下涉及到的点，当然目前肯定不全，后续遇到其他的知识点也会慢慢补齐。

## 一些关于Maven的知识点

 ### 生命周期

​	Maven将构建和分发项目的过程进行了明确的定义，这个就是Maven中的生命周期。一般默认的有以下几种：

- `validate` 验证项目正确，并获取所有必要信息
- `compile`编译项目源代码
- `test`使用合适的单元测试框架测试编译后的源代码。这些测试不应要求打包或部署代码
- `package` 取编译后的代码，并以可分发格式打包，例如JAR
- `verify`对集成测试结果进行任何检查，以确保满足质量标准
- `install`将软件包安装到本地存储库中，用于本地其他项目中的依赖项
- `deploy`在构建环境中完成，将最终的软件包复制到远程存储库中，以便与其他开发人员和项目共享

​	看着是不是特别熟悉，Idea中的Maven也有类似的。

![image-20210902220700371](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902220700371.png)

​	是不是也知道了Idea中这些按钮的含义。也知道了自己以后应该在什么步骤执行点击什么按钮了吧。

### Maven文件结构介绍

​	这一部分就是Maven对于项目各种文件的一种限制，不一定非要你完全执行，只是一个建议，和我们平时的项目结构差不多，和我上面的截图类似。具体含义如下：

| 目录                 | 存放文件类型                   |
| -------------------- | ------------------------------ |
| `src/main/java`      | 应用程序代码                   |
| `src/main/resources` | 资源库资源                     |
| `src/main/filters`   | 资源筛选文件                   |
| `src/main/webapp`    | 网络应用程序资源               |
| `src/test/java`      | 测试代码                       |
| `src/test/resources` | 测试需要资源                   |
| `src/test/filters`   | 测试资源筛选文件               |
| `src/it`             | 集成测试（主要用于插件）       |
| `src/assembly`       | 汇编描述符                     |
| `src/site`           | 站点                           |
| `LICENSE.txt`        | 项目许可证                     |
| `NOTICE.txt`         | 项目所依赖的库需要的通知和归属 |
| `README.txt`         | 项目介绍                       |

​	还有一些文件没有列出来，例如git、svn的文件，但是大致的结构都差不多。现在是不是清楚了为什么GitHub上下载的代码都是一个结构了吧。

###  原型简单介绍

​	上面简单Demo中使用到了一个原型，原型就是Maven为了方便我们创建项目而创建的基础项目。我们按照这个原型创建自己的项目。Maven的原型挺多的，当然，我们也可以自己定制自己的原型。

|          原型工件           |                             描述                             |
| :-------------------------: | :----------------------------------------------------------: |
|  Maven-archetype-archetype  |                   生成示例原型项目的原型。                   |
| Maven-archetype-j2ee-simple |               生成简单示例J2EE应用程序的原型。               |
|    Maven-archetype-mojo     |              用于生成示例Maven插件示例的原型。               |
|   Maven-archetype-plugin    |                 生成示例 Maven 插件的原型。                  |
| Maven-archetype-plugin-site |                生成Maven插件站点示例的原型。                 |
|   Maven-archetype-portlet   |               生成JSR-268 Portlet样本的原型。                |
| Maven-archetype-quickstart  |                  生成Maven项目示例的原型。                   |
|   Maven-archetype-simple    |                 生成简单 Maven 项目的原型。                  |
|    Maven-archetype-site     | 一个原型，用于生成Maven站点示例，演示一些受支持的文档类型，如APT、XDoc和FML，并演示如何i18n您的站点。 |
| Maven-archetype-site-simple |                  生成Maven站点样本的原型。                   |
|   Maven-archetype-webapp    |               生成Maven Webapp项目示例的原型。               |

​	可以按照自己的需求下载对应的原型，然后进行改造和开发。具体的介绍地址在这里：[Maven原型介绍](https://Maven.apache.org/guides/introduction/introduction-to-archetypes.html)

###  pom文件介绍

​	Maven项目最重要的就是Pom文件，但是Pom文件里面的内容实在太多了，我一篇博客很难讲完，这里偷个懒，讲Maven的介绍和参考链接放上来，也算是防止自己忘记吧。还是要说一句，任何的教程和文档都没有官方文档讲的详细。

[Maven官方Pom文件参考](https://Maven.apache.org/pom.html)

​	其实我们常用的就是本项目的坐标配置和项目需要的依赖。再顶多配置一下远程仓库、插件。细看这些配置就可以满足日常开发需求了。

## 开发中常用的功能

​	最后一个环节介绍一下平时开发中会用到的功能。也算比较常见吧。

### 配置私服地址,从私服下载依赖

​	大部分公司都搭建了私服，也就是边缘端端中心仓库。你的项目要想从这些仓库下载地址也很简单，有以下办法。

* 修改你的Maven的配置文件settings.xml,如下：

```xml
<settings>
    <profiles>
        <profile>
            <id>myprofile</id>
            <repositories>
                <repository>
                    <id>my-repo2</id>
                    <name>your custom repo</name>
                    <url>http://jarsm2.dyndns.dk</url>
                </repository>
            </repositories>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>myprofile</activeProfile>
    </activeProfiles>
</settings>
```

* 修改你的项目文件（会覆盖上一步的配置，所以是常用做法）

```xml
<repositories>
    <repository>
        <id>my-repo1</id>
        <name>your custom repo</name>
        <url>http://jarsm2.dyndns.dk</url>
    </repository>
    <repository>
        <id>my-repo2</id>
        <name>your custom repo</name>
        <url>http://jarsm2.dyndns.dk</url>
    </repository>
</repositories>
```

### pom中增加构建配置

​	通过配置profiles标签，来达到选定打包文件的操作，idea中的显示也比较方便，如下图：

![image-20210902223704130](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20210902223704130.png)

​	具体的代码看自己项目定。	

### 上传jar包到私服

​	这个步骤也是简单，首先在项目增加配置，你要上传的仓库的地址。再在你的Maven全局配置文件里面配置私服的用户名和地址，也可以配置其他验证方式。配置完成后，执行deploy命令即可。这里就不再详细的展示步骤了，不清楚的可以看看官网的介绍。

[部署到远程仓库](https://Maven.apache.org/guides/getting-started/index.html#How_do_I_deploy_my_jar_in_my_remote_repository)

​	但是需要注意，开发版本和稳定版本是有区别的，有时某些远程仓库是只能上传开发版本的，有些是只能上传稳定版本的。需要你上传时注意。

### 打包到本地仓库

​	这个就更加简单了，就是本地install命令即可，在仓库中寻找对应目录的jar即可。我这里也不演示了。

​	至此，Maven的介绍和一些我能想到的点就全部介绍完毕了。后续有机会还是会继续更新的。

​	就这样吧，结束。

