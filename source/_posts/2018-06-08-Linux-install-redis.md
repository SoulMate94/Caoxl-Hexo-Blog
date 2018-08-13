---
title: Linux 安装redis
date: 2018-06-08 11:06:52
categories: Linux
tags: [Linux, Redis]
---

> `redis`是用C语言开发的一个开源的高性能键值对（key-value）数据库。它通过提供多种键值数据类型来适应不同场景下的存储需求，目前为止redis支持的键值数据类型如下
字符串、列表（lists）、集合（sets）、有序集合（sorts sets）、哈希表（hashs）

<!-- more -->

# `Redis`的安装和部署

## 安装/部署

- 下载

```linux
    wget http://download.redis.io/releases/redis-4.0.0.tar.gz
```

**在http://download.redis.io/releases/找自己需要的版本**

- 移动

```linux
    cp redis-4.0.0.rar.gz /usr/local
```

- 解压并进入目录

```linux
    tar -zxvf redis-4.0.0.tar.gz
    cd /usr/local/redis-4.0.0 
```

- 编译安装

```linux
    // 安装到指定目录
    make PREFIX=/usr/local/redis install
```

**redis.conf是redis的配置文件，redis.conf在redis源码目录**

- 拷贝配置文件

```linux
    cd /usr/local/redis
    mkdir conf
    cp /usr/local/redis-4.0.0/redis.conf  /usr/local/redis/bin
```

- 进入安装目录bin下

```linux
    cd /usr/local/redis/bin
```

**目录结构解释:**
- `redis-benchmark`: redis性能测试工具
- `redis-check-aof`: AOF文件修复工具
- `redis-check-rdb`: RDB文件修复工具
- `redis-cli`: redis命令行客户端
- `redis.conf`: redis配置文件
- `redis-sentinal` redis集群管理工具
- `redis-server`: redis服务进程

## 启动redis

- 前端模式启动

```linux
    cd /usr/local/redis/bin/
    ./redis-server
```

- 后台模式启动

修改redis.conf配置文件， daemonize yes 以后端模式启动

```linux
    vim /usr/local/redis/bin/redis.conf
    daemonize yes
```

执行如下命令启动redis：

```linux
    cd /usr/local/redis/bin
    ./redis-server ./redis.conf
```

连接`redis`

```linux
    /usr/local/redis/bin/redis-cli
```

## 关闭`redis`

```
    shutdown
    
    // 强行终止redis
    pkill redis-server
```

## `redis`开机启动

```
    vim /etc/rc.local
    //添加
    /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis-conf
```


# `Redis`扩展的安装

## 下载

```
    wget http://pecl.php.net/get/redis-4.1.0.tgz // 版本自选
```

## 解压/进入目录

```
    tar zxf redis-4.1.0.tgz 
    cd redis-4.1.0
```

## 在Redis文件夹下，生成configure配置文件

```
    /usr/local/php/bin/phpize
    ./configure --with-php-config=/usr/local/php/bin/php-config
    make && make install
```

`redis.so`扩展存放在`/usr/local/php/lib/php/extensions/no-debug-non-zts-20160303/`目录下。

## 在PHP配置文件php.ini里面加载Redis扩展

```
    extension=redis.so
```

## 重启服务器(Apache或者Nginx)

```
    service httpd restart
    // 或
    service nginx start
```

## 测试

浏览器`phpinfo`信息，如果有Redis信息，则安装成功

# 参考

- [Linux下redis安装和部署](https://www.jianshu.com/p/bc84b2b71c1c)
- [Linux下PHP安装Redis扩展（二）](https://segmentfault.com/a/1190000008420258)