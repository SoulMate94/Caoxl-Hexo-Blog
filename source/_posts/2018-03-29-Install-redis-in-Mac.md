---
title: Mac下安装配置Redis
date: 2018-03-29 16:54:22
categories: Redis
tags: Redis
---


> 一、准备工作
  二、安装redis服务器
  三、redis服务器的启动、使用和退出
  四、配置php使用redis服务
  五、常见问题

<!-- more -->

# 一 准备工作

- 安装homebrew
- 安装php、nginx（或apache）或集成环境

# 二 安装redis服务器

- 通过homebrew安装

```php
    brew install redis
```

- 直接下载安装

```php
    curl -O http://redis.googlecode.com/files/redis-2.8.7.tar.gz
    sudo tar -zxf redis-2.8.7.tar.gz
    mv redis-2.8.7 /usr/local/redis
    cd redis
    sudo make
    sudo make test
    sudo make install
    mv redis.conf /etc/redis.conf
```

# 三 redis服务器的启动、使用和退出

- 1 启动redis服务

```php
    /usr/local/bin/redis-server
```

- 2 查看redis服务是否启动

```php
    ps aux | grep redis
```

- 3 使用redis服务

通过redis-cli命令可以启动redis客户端

```php
    redis-cli
```

常用命令见：上一篇文章

- 4 退出redis服务
  - 客户端退出

  ```php
        redis-cli shutdown
  ```

  - 关闭pid

  ```php
        ps -u jim(替换成你的用户名) -o pid,rss,command | grep redis-server
  ```

**如果你的电脑安装了oh my zsh**

那么只需要在终端输入

```php
    kill redis
```

按tab，会自动替换成对应的pid

再运行:

```php
    kill -9 对应的pid
```


# 四 配置php使用redis服务

- 安装php的redis扩展

```php
    brew install php71-redis --build-from-source
```

`php71`是本机安装的php的版本（7.1）,`--build-from-source`是让安装的扩展与php的版本保持一致


查看`phpinfo()`，出现redis选项说明redis配置成功

- 在php代码中使用redis服务

```php
    $redis = new Redis();
    $redis->connect('127.0.0.1','host');//redis服务器ip及端口号
    $redis->set($key,$value,$timeout);//设置缓存:键-值-缓存时间
    $redis->get($key);//查找缓存
    $redis->del($key);//删除缓存
    $redis->delete($key);//删除缓存
```


# 五 常见问题


- （1）Redis: Failed opening .rdb for saving: Permission denied

redis服务器会生成`dump.rdb`文件存储缓存，如果文件权限不够则无法读写该文件

```php
    cd /usr/loal/bin
```

在`/usr/local/bin/`（默认文件目录）下执行命令

```php
    chmod 777 dump.rdb
```