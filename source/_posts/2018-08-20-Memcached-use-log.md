---
title: Memcached 使用日志
date: 2018-08-20 15:06:21
categories: 缓存
tags: [Cache, Memcached]
---

> `Memcached` 在正式环境的安装+使用日志。

<!-- more -->

# `memcached` 服务器

## 虚拟主机／CentOS 7 运行

```
    # 安装配置
    yum update && yum install -y memcached
    # SASL 支持
    yum install cyrus-sasl-devel cyrus-sasl-plain
    
    # 启动+服务自启
    memcached -d -p 11211 -u memcached -m 229 -c 1024 -P /var/run/memcached/memcached.pid -l 127.0.0.1
    systemctl start memcached
    systemctl enable memcached
    
    # 查看运行状态
    netstat -plunt
    memstat --servers="127.0.0.1"
    
    # 允许外网访问（慎用）
    firewall-cmd --add-port=11211/tcp --permanent
    firewall-cmd --reload
```

除了通过命名指定 memcached 的启动参数外，最好的方式是将通过配置文件启动：

```
    vi /etc/sysconfig/memcached
    
    `
    PORT="11211"
    USER="memcached"
    MAXCONN="1024"
    CACHESIZE="64"
    OPTIONS="-l 127.0.0.1 -U 0" 
    `
```

### 关于服务启动选项

- `-d` 是启动一个守护进程
- `-m` 是分配给Memcache使用的内存数量，单位是MB
- `-u` 是运行Memcache的用户
- `-l` 是监听的服务器IP地址，可以有多个地址
- `-p` 是设置Memcache监听的端口，最好是 1024 以上的端口
- `-c` 是最大运行的并发连接数，默认是1024
- `-P` 是设置保存 Memcache 的 pid 文件

## Docker 容器运行

```
    docker run --name local-memcached -d -p 11211:11211 \
    -e VIRTUAL_HOST=memcached.docker \
    -h memcached.docker \
    memcached
```

# `memcached` 客户端

## CLI 使用 memcached

### telnet

```
    telnet 127.0.0.1 11211
```

终止 telnet：

- ctrl + ]
- quit\r\n

### netcat/nc

```
    nc 127.0.0.1 11211
```

### memcache-cli

- `python`

```
    pip install memcache-cli
    memcache-cli host1:port host2:port  # 无参数默认 127.0.0.1:11211
    
    # macOS
    brew upgrade python
    pip2 install --upgrade pip
    pip2 install memcache-cli
```

- `nodejs`

```
    # npm 
    npm install -g memcached-cli
    # yarn 
    yarn global add memcached-cli
    
    memcached-cli host:port
    memcached-cli username:password@host:port
```

## PHP 使用 memcached

### 安装 memcached 扩展完整流程

源码安装 memcached 扩展并启用 `SASL`／`igbinary`／`json` 完整步骤：

```
    # 确保 `gcc -v` 大于 4.2+
    # 否则：yum install gcc+ gcc-c++
    
    # 确保 php 完整环境已安装
    # 否则：yum install php-devel php-common php-cli
    
    # 确保 SASL 相关环境包已正确安装
    # 否则：yum install cyrus-sasl-plain cyrus-sasl cyrus-sasl-devel cyrus-sasl-lib
    
    # 确保 libmemcached 已安装
    # rpm -qa | grep libmemcached
    # 否则下载编译安装
    wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
    tar zxf libmemcached-1.0.18.tar.gz
    cd libmemcached-1.0.18
    ./configure --prefix=/usr/local/libmemcached --enable-sasl
    make && make install
    
    # 安装 igbinary 扩展
    # yum install -y php-pecl-igbinary
    wget https://github.com/igbinary/igbinary/releases/download/2.0.5/igbinary-2.0.5.tgz
    tar -xzvf igbinary-2.0.5.tgz
    cd igbinary-2.0.5
    phpize
    ./configure
    make && make install
    
    ` ini
    extension=igbinary.so
    `
    
    # 安装 memcached
    yum install zlib-devel
    wget http://pecl.php.net/get/memcached-3.0.4.tgz
    tar zxf memcached-3.0.4.tgz
    cd memcached-3.0.4
    phpize    # 如果有多套 PHP 环境则需要使用各自的绝对路径
    ./configure --with-libmemcached-dir=/usr/local/libmemcached --enable-memcached-sasl --enable-memcached-json --enable-memcached-json --enable-memcached-igbinary
    make && make install
    cp modules/memcached.so /path/to/php_extensions/no-debug-non-zts-XXXXXX/
    
    # 编辑 /path/to/php.ini
    ` ini
    extension=memcached.so
    memcached.use_sasl = 1
    `
    
    # 重启 php-fpm
    service php-fpm restart
```

### 代码示例

```
    $connect= new Memcached; //声明一个新的memcached链接
    $connect->setOption(Memcached::OPT_COMPRESSION, false); //关闭压缩功能
    $connect->setOption(Memcached::OPT_BINARY_PROTOCOL, true);//使用binary二进制协议
    $connect->setOption(Memcached::OPT_TCP_NODELAY, true); //重要，php memcached有个bug，当get的值不存在，有固定40ms延迟，开启这个参数，可以避免这个bug
    $connect->addServer('xxxxxxxx.m.yyyyyyyy.ocs.aliyuncs.com', 11211);//添加OCS实例地址及端口号
    $connect->setSaslAuthData('xxxxxxxx', 'bbbbbbbb');//设置OCS帐号密码进行鉴权，如已开启免密码功能，则无需此步骤
    $user = array(
        "name" => "ocs",
        "age" => 1,
        "sex" => "male"
    ); //声明一组数组  
    $expire = 60; //设置过期时间
    $connect->set('your_name',$user,$expire);
```

# FAQ

## PHP `addServer()` “成功”后 `getStats()` 返回 false

- 检查 memcached 安装配置是否正确
- 检查是否 SASL 连接配置是否正确。

# 附录：memcached 命令

<u>[memcached Cheat Sheet。](https://lzone.de/cheat-sheet/memcached)</u>

# 参考

- [Docker Hub-Memcached](https://hub.docker.com/_/memcached/)
- [ How To Install and Secure Memcached on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-and-se1cure-memcached-on-centos-7)
- [memcachecli](https://github.com/andrewgross/memcache-cli/tree/master/memcachecli) && [memcached-cli](https://www.npmjs.com/package/memcached-cli)
- [阿里云文档-PHP: memcached](https://help.aliyun.com/document_detail/48432.html?spm=a2c4g.11186623.4.5.ROL7iq)
- [libmemcached-memcached 依赖](https://launchpad.net/libmemcached)
- [memcache及其telnet命令使用详解](http://jony-hwong.iteye.com/blog/1174985)

