---
title: Docker 初见
date: 2018-01-04 18:29:39
categories: DevTool
tags: [Docker, DevTool]
---

## Docker-First-Try

> 因为PHP开发环境需要多个环境,由此学习Docker


最近的一个项目使用了低版本的 PHP `5.4`，而我主要用的 `PHP5.6`，又不时想体验最新的 `PHP 7+`，之前一直使用 `vagrant` 管理 `Linux 虚拟机，里面同一时刻只能使用一个版本的 PHP，虽然可以安装多个版本的虚拟机，但是每次都要更新环境变量，且同一时刻只能运行一个项目。

<!--more-->

> 下面以在 macOS 上总结使用 Docker 运行 N 个版本项目的实际经历，以及学到的新东西。

## 安装 Docker

`Docker` 可以直接安装在 `Linux`、`macOS`，甚至 `Windows` 上。

这里有一个 `Docker Toolbox` 的东西，如果安装的是这个，那么无论是在 `macOS` 还是 `Windows` 上，`Docker` 实际运行的系统还是 `Linux`，因为安装 `Docker Toolbox` 的时候会安装好 `VirtualBox` 和一个 `Linux` 虚拟机。

推荐直接安装 [Docker for Mac](https://docs.docker.com/docker-for-mac/install/)，这样可以不用绕几个弯，直接让 Docker 运行在 macOS 上。

## Docker 镜像

> Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资 源、配置等文件外，还包含了一些为运行时准备的一些配置参数(如匿名卷、环境 变量、用户等)。镜像不包含任何动态数据，其内容在构建之后也不会被改变。


### 设置加速镜像

至于为什么要先设置这个，因为不设置国内镜像下载速度真的过于感人，不方便后面的操作。（神秘的东方力量，大家都懂的）


国内有阿里云和 DaoCloud 等大厂提供的 Docker 加速镜像地址，我这里使用的是 DaoCloud，在[配置 Docker 加速器](https://www.daocloud.io/mirror#accelerator-doc)这里，有文档说明在三大操作系统中镜像地址的配置。以 Docker For Mac 为例：

> 右键点击桌面顶栏的 docker 图标，选择 Preferences ，在 Daemon 标签（Docker 17.03 之前版本为 Advanced 标签）下的 Registry mirrors 列表中加入下面的镜像地址: http://xxxx.m.daocloud.io。


### 下载镜像
```
    # 完整命令格式：docker pull [OPTIONS] [DOCKER_REGISTRY] <IMAGE_NAME>[:TAG|@DIGEST]
    docker pull nginx
```

DOCKER_REGISTRY 是镜像的来源，格式：<域名/IP>[:端口]，不指明则默认为 Docker Hub。

IMAGE_NAME 是镜像在镜像源上的仓库名，格式：[用户名/]<软件名>，如果不指定用户名则默认使用官方镜像用户名 Library。

这里将从 Docker Hub 下载 `library/nginx:latest`。


### 定制镜像
定制镜像有两种方式：
 - commit
 ```
    docker commit \
    --author "AUTHOR NAME" \
    --message "COMMIT MESSAGE" \
    BASE_IMAGE \
    NEW_IMAGE[:NEW_TAG]
 ```
 
 - Dockerfile
 ```
    docker build -t IMAGE[:TAG] .
 ```
 
但是，**不建议使用 commit 方式定制镜像，** 原因如下：
使镜像非常庞大：
> 在构建镜像过程中，除了命令的执行，还有很多文件被改动或添加了。这还仅仅是最简单的操作，如果是安装软件包、编译构建，那会有大量的无关内容被添加进来，如果不小心清理，将会导致镜 像极为臃肿。

不方便共享和维护：
> 此外，使用 docker commit 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为黑箱镜像，换句话说，就是除了制作镜像的人知道执行过什么命令、怎 么生成的镜像，别人根本无从得知。而且，即使是这个制作镜像的人，过一段时间 后也无法记清具体在操作的。虽然 docker diff 或许可以告诉得到一些线索， 但是远远不到可以确保生成一致镜像的地步。这种黑箱镜像的维护工作是非常痛苦 的。


### 镜像常用命令
 - 删除所有未打 tag 的镜像：docker rmi $(docker images -q | awk '/^<none>/ { print $3 }')
 - 删除所有镜像：docker rmi $(docker images -q)
 - **上传一个自定义镜像到 Docker Hub：**
 ```
    docker tag IMAGE[:TAG] NAMESAPCE/IMAGE[:TAG]
    docker push NAMESAPCE/IMAGE[:TAG]
 ```
 
 这里如果上传失败，则需要再 hub.docker.com 注册一个 namespace，然后使用 docker login 登录后，再试一下。
 
 上传时如果已经 tag 存在，则会直接更新。

## Docker 容器
### 镜像和容器的关系？
都是学过 OOP 的，我想关于 Docker 中镜像和容器的区别用 OOP 中的一句很基本的概念来类比就是：镜像相当于类，容器则相当于对象。我觉得够了，不多作解释。

### 启动容器

```
    docker run --name nginx-container -d -p 8888:80 --rm=true nginx
```
- `docker run` 将从一个镜像中启动一个容器。
- `--name` 为这个容器命名
- `-d` 表示 后台运行
- `-p` 指明端口映射关系，这里的 8888:80 代表的是将宿主系统上来自 8888 端口的数据转发到 nginx-container 这个容器的 80 端口。
- `--rm=true` 表示容器运行结束时自动删除该容器。
- `nginx` 表示要启动 nginx-container 这个容器的镜像名。

### 容器常用命令
 - 启动容器并打开 Bash：`docker run -it --rm=true IMAGE[:TAG] /bin/bash`
 - 删除所有 Docker 容器：`docker rm $(docker ps -a -q)`
 - 删除所有已退出运行的容器：`docker rm $(docker ps -qf status=exited)`
 
## 注意事项

### 路径问题

可能大家在初次构建镜像的时候，都会遇到提示找不到文件或者路径的提示，导致构建失败。下面我把我遇到的几种情况简单总结一下：

 - `ADD`
 比如：ADD ./build /root 是把当前路径下 build 文件夹下所有的文件和目录添加到容器的 *root* 路径中，是不再含 *build* 的。因此在容器或者 Dockerfile 中不能通过 */root/build/file* 来引用，而是 */root/file*。
 
 - `WORKDIR`
 这个指定了工作路径，在构建的时候所有的相对路径会以这个路径为基础进行引用，同时，构建结束之后，进入容器时默认也会进入这个路径。
 
 
## 实战：构建一个 CentOS 6+Apache+PHP 镜像
下面就为最近做的那个低版本 PHP（5.4）项目来构建的一个专门的镜像。

准备好如下文件：
```
    cd /path/to/php54
    tree .
    .
    ├── Dockerfile
    ├── README.md
    ├── app
    │   └── index.php
    ├── build
    │   ├── ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64.tar.gz
    │   └── zendguardloader.ini
    └── docker.run
    2 directories, 6 files
```

 - Dockerfile 的内容：
 ```
    FROM centos:6
    MAINTAINER Caoxl <code0809@163.com>
    ADD ./build /tmp
    WORKDIR /tmp
    RUN yum update -y \
    	&& yum install -y wget\
    	&& mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup \
    	&& wget -O /etc/yum.repos.d/CentOS-Base.repo  http://mirrors.aliyun.com/repo/Centos-6.repo \
    	&& yum install -y epel-release \
    	&& wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm \
    	&& rpm -Uvh remi-release-6*.rpm \
    	&& sed -i 's/enabled=0/enabled=1/g' /etc/yum.repos.d/remi-php54.repo \
    	&& yum install -y php php-mysql php-pdo\
    	
    	# && wget http://downloads.zend.com/guard/6.0.0/ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64.tar.gz \
    	# && tar zxf ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64.tar.gz \
    	# && cp ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64/php-5.4.x/ZendGuardLoader.so /usr/lib64/httpd/modules/ \
    	# && echo 'zend_extension=/usr/lib64/httpd/modules/ZendGuardLoader.so' >> /etc/php.ini \
    	# && echo 'zend_loader.enable=1' >> /etc/php.ini \
    	# && echo 'zend_loader.disable_licensing=0' >> /etc/php.ini \
    	# && echo 'zend_loader.obfuscation_level_support=3' >> /etc/php.ini \
    	&& tar zxf ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64.tar.gz \
    	&& cp ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64/php-5.4.x/ZendGuardLoader.so /usr/lib64/httpd/modules/ \
    	&& cat zendguardloader.ini >> /etc/php.ini \
    	&& echo 'ServerName localhost' >> /etc/httpd/conf/httpd.conf \
    	
    	&& yum clean all \
    	&& rm -rf /tmp/* \
    	&& rm -rf /root/* \
    	&& rm -rf /var/tmp/*
    ADD ./app /var/www/html
    ADD ./docker.run /root
    RUN chmod 755 /root/docker.run
    WORKDIR /var/www/html
    EXPOSE 80
    CMD ["/bin/bash", "/root/docker.run"]
 ```
 
 - README.md
 ```
    - build
    `docker build -t php54:v1 .`
    - run
    `docker run -it --name php54 --rm=true -d -p 80:80 php54:v1 /var/www/html/docker.run`
    - test
    <http://localhost>
 ```
 
 - docker.run
 ```
    #!/bin/bash
    /usr/sbin/apachectl -D FOREGROUND
 ```
 
 - app/index.php
 ```
    <?php
    echo '<h1 style="text-align:center">Hello Docker</h1>';
    phpinfo();
 ```
 
 - build/ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64.tar.gz
 
 从 http://downloads.zend.com/guard/6.0.0/ZendGuardLoader-70429-PHP-5.4-linux-glibc23-x86_64.tar.gz 下载而来。
 
 - build/zendguardloader.ini
 ```
    zend_extension=/usr/lib64/httpd/modules/ZendGuardLoader.so
    zend_loader.enable=1
    zend_loader.disable_licensing=0
    zend_loader.obfuscation_level_support=3
 ```
 
 最后构建：
 ```
    cd /path/to/php54
    docker build -t php54:v1 .
 ```
 
 如果不出意外，build 结束之后在 docker images 中就可以看到新建的镜像了，便可以运行：
 ```
    docker run -it --name php54 --rm=true -d -p 80:80 php54:v1
 ```
 
 然后打开浏览器：http://localhost 正常情况即可看到 app/index.php 的输出。
 
 同理，如果需要使用 PHP5.6／7 版本的环境，可以按照同样的步骤 build 一个相应的镜像，启动后，可以通过本机的不同端口去访问，这样就实现了在一个系统上运行 N 个版本项目的需求。
 
 Cool~ isn’t it ? (:
 
 
## 参考
- [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/)
- [PHP Web Development with Docker](http://mmenozzi.github.io/2016/01/22/php-web-development-with-docker/)
- [PHP 开发者的 Docker 之旅](http://guide.daocloud.io/dcs/php-docker-9153862.html)
- [Docker Documentation](https://docs.docker.com/)
- [Oracle VM VirtualBox-User Manual](https://www.virtualbox.org/manual/UserManual.html)
- [Why docker container exits immediately](https://stackoverflow.com/questions/28212380/why-docker-container-exits-immediately)
- [Restart apache on Docker](https://stackoverflow.com/questions/36869443/restart-apache-on-docker)
- [Centos/RedHat 7/6/5切换阿里云源并安装EPEL/IUS/REMI仓库](https://blog.kuoruan.com/65.html)
- [CentOS 安装 Zend Guard Loader](https://blog.itnmg.net/2013/07/05/centos-zend-guard-loader/) Or [Install Zend Guard 6.0 with PHP 5.4 on Centos](https://briansnelson.com/Install_Zend_Guard_6.0_with_PHP_5.4_on_Centos)
- [CentOS Yum 命令详解](http://desert3.iteye.com/blog/1664813)
- [将Centos的yum源为国内的阿里云源](http://blog.csdn.net/codemanship/article/details/45056977)
- [denied: requested access to the resource is denied : docker](https://stackoverflow.com/questions/41984399/denied-requested-access-to-the-resource-is-denied-docker)
