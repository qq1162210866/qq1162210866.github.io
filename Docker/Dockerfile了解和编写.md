## Dockerfile了解和编写

​	这个也是之前欠下的技术帐，也是慢慢补上吧，算是对于Docker的更加深入的理解和应用。

​	Docker非常好用，简单快捷。但是如果想要自己构建一个独特的镜像，简单的使用显然不能够完成，需要更加深入的理解。这篇博客就是记录这个事情的。下面就开始吧。

​	在学习编写Dockerfile文件前，需要先了解Docker中镜像的构成。

​	一个镜像最开始是由父镜像生成的，这个则是由Dockerfile中`FROM`指令指定的。例如`FROM ubuntu:18.04`就是代表这个镜像是基于ubuntu镜像构建的。Docker中的镜像是由多个镜像层构成的，例如你下载的时候就会显示多个镜像层。如下图：

![image-20201120103701404](https://1162210866.oss-cn-beijing.aliyuncs.com/uPic/image-20201120103701404.png)

一行就代表一个镜像层，可以看到这个镜像有8个镜像层。下载也是分别下载镜像层，这样做的好处是可以减少下载的流量，有可能不同的镜像用的是同一个镜像层。有时候你删除某个镜像，如果它依赖于java镜像，可能java镜像并不会删除。镜像层的概念就类似于此。为什么一个镜像会有多层的概念，这时就要讲到Dockerfile文件命令了，Dockerfile文件中有多少个命令，构建的镜像就会有多少层镜像层。例如下面的Dockerfile文件就会构建一个有三层的镜像。

```dockerfile
FROM scratch
ADD hello /
CMD ["/hello"]
```

​	介绍完了简单的镜像的一些知识，下面就开始编写Dockerfile文件。Dockerfile文件通过由以下命令组成。FROM、MAINTAINER、RUN、CMD、EXPOSE、ENV、ADD、COPY、ENTRYPOINT、VOLUME、USER、WORKDIR、ONBUILD。这些命令的使用组成了一个Dockerfile文件。再简单举个例子。这里以Docker官方的ruby文件为例。

```dockerfile
FROM buildpack-deps:buster
# skip installing gem documentation
RUN set -eux; \
	mkdir -p /usr/local/etc; \
	{ \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /usr/local/etc/gemrc
ENV LANG C.UTF-8
ENV RUBY_MAJOR 3.0-rc
ENV RUBY_VERSION 3.0.0-preview1
ENV RUBY_DOWNLOAD_SHA256 aa7cce0c99f4ea2145fef9b78d74a44857754396790cd23bad75d759811e7a2a
# some of ruby's build scripts are written in ruby
#   we purge system ruby later to make sure our final image uses what we just built
RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		bison \
		dpkg-dev \
		libgdbm-dev \
		ruby \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
	wget -O ruby.tar.xz "https://cache.ruby-lang.org/pub/ruby/${RUBY_MAJOR%-rc}/ruby-$RUBY_VERSION.tar.xz"; \
	echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.xz" | sha256sum --check --strict; \
	\
	mkdir -p /usr/src/ruby; \
	tar -xJf ruby.tar.xz -C /usr/src/ruby --strip-components=1; \
	rm ruby.tar.xz; \
	\
	cd /usr/src/ruby; \
	\
# hack in "ENABLE_PATH_CHECK" disabling to suppress:
#   warning: Insecure world writable dir
	{ \
		echo '#define ENABLE_PATH_CHECK 0'; \
		echo; \
		cat file.c; \
	} > file.c.new; \
	mv file.c.new file.c; \
	\
	autoconf; \
	gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	./configure \
		--build="$gnuArch" \
		--disable-install-doc \
		--enable-shared \
	; \
	make -j "$(nproc)"; \
	make install; \
	\
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark > /dev/null; \
	find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	\
	cd /; \
	rm -r /usr/src/ruby; \
# verify we have no "ruby" packages installed
	! dpkg -l | grep -i ruby; \
	[ "$(command -v ruby)" = '/usr/local/bin/ruby' ]; \
# rough smoke test
	ruby --version; \
	gem --version; \
	bundle --version
# don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_SILENCE_ROOT_WARNING=1 \
	BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $GEM_HOME/bin:$PATH
# adjust permissions of a few directories for running "gem install" as an arbitrary user
RUN mkdir -p "$GEM_HOME" && chmod 777 "$GEM_HOME"
CMD [ "irb" ]
```

​	可以看到上面的例子使用了四种指令，分别为`FROM`,`RUN`,`ENV`,`CMD`。`FROM`的作用就是声明基础镜像是哪个，也就是本次构建的镜像是由以哪个镜像为基础构建的。`RUN`常用就是安装软件，需要注意的就是一条指令会创建一个镜像层，所以尽可能减少多个`RUN`,让一个指令做更多的事，减少镜像层的臃肿。上面的例子也是如此，可以看到大量的用了`\`，就是为了避免创建过多的镜像层（比较考验Linux的功底）。`ENV`就是设置环境变量，需要注意的是，和Java中的类变量类似，上一层的镜像或者基于本镜像创建的镜像都可以使用这个环境变量。`CMD`是启动容器的时候被执行的命令，一个Dockerfile文件中只有一个`CMD`,可以有多个`RUN`.理解了这几个指令，再看上面的例子就能看出大概了，例子中先是声明使用了哪个基础镜像，再输入若干`RUN`和`ENV`指令去构建镜像，最好则是`CMD`指令，代表容器启动的时候镜像执行的命令。这样一个镜像的Dockerfile文件就写完了。

​	下面就开始执行`docker build`,构建镜像，然后就可以看到docker中有刚才构建的镜像了。构建镜像的步骤就算结束了，后续如果还有的话就是将镜像推送到云上的仓库，下一篇博客可能会讲一讲。

​	关于这些指令的详细介绍大家可以看看别的博客的介绍或者官网上的介绍（英文的）。后面就是官方的地址：[https://docs.docker.com](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#dockerfile-instructions) 。后续我也会慢慢的补充，慢慢来吧。

​	就这样吧，结束。