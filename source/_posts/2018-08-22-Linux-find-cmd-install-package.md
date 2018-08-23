---
title: Linux查找命令或组件对应安装包的方法
date: 2018-08-22 11:24:17
categories: Linux
tags: [Linux]
---

当新搭建服务器或者维护不熟悉的服务器环境时，难免会碰到自己想用的命令或组件不存在的情况。如何通过包管理软件，方便地找到命令或组件对应的package进行安装？下面介绍三种方法。

<!-- more -->

# 名称搜索

一种直观方法就是，猜测命令或组件与安装包同名或包含，尝试搜索安装。比如，想安装redis数据库：

```
    [root@caoxl ~]# yum search redis
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
    ===================================== N/S matched: redis =====================================
    collectd-redis.x86_64 : Redis plugin for collectd
    collectd-write_redis.x86_64 : Redis output plugin for collectd
    hiredis.x86_64 : Minimalistic C client library for Redis
    hiredis-devel.x86_64 : Development files for hiredis
    opensips-redis.x86_64 : Redis connector
    pcp-pmda-redis.x86_64 : Performance Co-Pilot (PCP) metrics for Redis
    perl-Apache-Session-Redis.noarch : Redis driver for Apache::Session::NoSQL
    perl-Redis.noarch : Perl binding for Redis database
    php-nrk-Predis.noarch : PHP client library for Redis
    php-pecl-redis.x86_64 : Extension for communicating with the Redis key-value store
    php-phpiredis.x86_64 : Client extension for Redis
    python-redis.noarch : Python 2 interface to the Redis key-value store
    python-trollius-redis.noarch : Redis client for the Python event loop PEP3156 for Trollius.
    python2-django-redis.noarch : Full featured redis cache backend for Django
    redis-trib.noarch : Cluster management script for Redis
    rubygem-redis.noarch : A Ruby client library for Redis
    rubygem-redis-doc.noarch : Documentation for rubygem-redis
    syslog-ng-redis.x86_64 : redis support for syslog-ng
    uwsgi-logger-redis.x86_64 : uWSGI - redislog logger plugin
    uwsgi-router-redis.x86_64 : uWSGI - Plugin for Redis router support
    redis.x86_64 : A persistent key-value database
    
      Name and summary matches only, use "search all" for everything.
```

搜索repo的相关命令：

```
    # Centos 7
    yum search {cmd} 

    # Ubuntu 14.04
    apt-cache search {cmd} 
```

# command-not-found组件：自动提示命令

`command-not-found`组件，支持在命令未找到时，提示对应的安装命令语句

command-not-found的安装方法：

```
    # Centos 7
    # yum search command-not-found
    yum install -y PackageKit-command-not-found 
    
    # Ubuntu 14.04(很可能已预装)
    # apt-cache search command-not-found
    apt-get install command-not-found 
```

# 文件反查安装包

果在身边其他的环境中，已经安装了相应的命令或组件，可以通过已安装的文件反查对应的安装包，如查找ss命令：

```
    [root@caoxl ~]# which ss
    /usr/sbin/ss
    [root@caoxl ~]# rpm -qf /usr/sbin/ss
    iproute-4.11.0-14.el7.x86_64
```

反查文件对应安装包的命令：

```
    # Centos 7
    rpm -qf {file}
     
    # Ubuntu 14.04
    dpkg -S {file}  
```

另外，`ubuntu`的`apt-file`命令也支持安装包包含的文件反查安装包，即使未安装该安装包。

> 当然，终极大招就是：互联网上搜索！