---
title: PHP 环境问题汇总
date: 2018-01-22 15:06:15
categories: PHP
tags: [PHP, Nginx, 配置]
---

> 在开发部署 PHP 项目过程中总会遇到一些环境相关的问题，这里简单记录下来。

<!--more-->

# `nginx` 连接 `php-fpm` 的两种方式

php-fpm 的启动方式有两种：

## TCP sockets

就是常见的，也是 php-fpm 默认的启动方式：**监听本机 9000 端口，通过 TCP 协议通信。**

## UNIX domain sockets

> Unix domain socket 或者 IPC socket是一种终端，可以使同一台操作系统上的两个或多个进程进行数据通信。与管道相比，Unix domain sockets 既可以使用字节流和数据队列，而管道通信则只能通过字节流。Unix domain sockets的接口和Internet socket很像，但它不使用网络底层协议来通信。Unix domain socket 的功能是POSIX操作系统里的一种组件。
>  
> Unix domain sockets 使用系统文件的地址来作为自己的身份。它可以被系统进程引用。所以两个进程可以同时打开一个 Unix domain sockets 来进行通信。不过这种通信方式是**发生在系统内核里而不会在网络里传播。**

## TCP vs UNIX Socket

> When you are using TCP, you are also using the whole network stack. Even if you are on the same machine, this implies that packets are encapsulated and decapsulated to use the network stack and the related protocols.
>  
> If you use unix domain sockets, you will not be forced to go through all the network protocols that are required otherwise. The sockets are identified solely by the inodes on your hard drive.
> 
> Beware though that sockets are only reachable from programs that are running on the same server (there’s no network support, obviously) and that the programs need to have the necessary permissions to access the socket file.

在服务器压力不大的情况下，tcp 和 socket 差别不大，但在压力比较满的时候，用套接字方式，效果确实比较好。

## php-fpm 配置

修改 `/usr/local/php/etc/php-fpm.conf` 文件，换掉注释：

```
    ;listen = 127.0.0.1:9000
    listen  = /dev/shm/php-cgi.sock
    listen.owner = www
    listen.group = www
```

由于权限问题很多人使用路径 `/tmp`，但路径 /`dev/shm` 是个 `tmpfs`，速度比磁盘快得多。

至于权限问题，可以在 `php-fpm.conf` 中指定所有者和所有组决定。否则生成的 `/dev/shm/php-cgi.sock` 是 `root` 所有的，`PHP` 没有权限访问。

最后重启 `php-fpm：service php-fpm restart`。可以查看是否成功：

```
    lsof -i:9000    # nothing
    
    ls -al /dev/shm
```


## nginx 配置

可以对虚拟主机的 `server` 字段进行配置：

```
    location ~ [^/]\.php(/|$) {
        #fastcgi_pass 127.0.0.1:9000;
        fastcgi_pass unix:/dev/shm/php-cgi.sock;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
    
    # 或者创建一个配置文件 php5-fpm.conf
    upstream php5-fpm-sock {
        server unix:/var/run/php5-fpm.sock;
    }
    
    # 引入 nginx 后在 nginx `Server` 字段中引入
    fastcgi_pass php5-fpm-sock;
```

最后重启 `nginx：service nginx restart`。


# 出现 “神奇的” `40x`/`50x` 错误 ？

- 安装的 `php-fpm` 版本和 `php -v` 是否一致？（500）
- Nginx 配置是否有误？路径是否存在？（403）
- CLI 下测试入口文件是否有报错？（502）
- `composer vendor` 目录是否安装完整？（autoload class）
- 检查错误日志（php.ini => php_errors）


# 扩展

## Xdebug

- xdebug must be loaded as a zend extension in unknown on line 0?

需要在 php.ini 中改变引入扩展的指令为 `zend_extension`：

```
    zend_extension = "xdebug.so"
```

然后重启 php-fpm 或者 apache。

```
    service php-fpm restart
    
    or:
    
    apache httpd restart
```

# FAQ

- **如何查看操作系统字符集？**

在 Windows 平台下，命令提示符中输入 `chcp` 可以得到操作系统的代码页信息，你可以从控制面板的语言选项中查看代码页对应的详细的字符集信息。

例如：我的活动代码页为 `936`，它对应的编码格式为 `GBK`。（简体中文的 Windows 字符编码是 GBK）


# 参考

- [使用socket方式连接Nginx优化php-fpm性能](https://blog.linuxeye.cn/364.html)
- [Performance of unix sockets vs TCP ports](https://unix.stackexchange.com/questions/91774/performance-of-unix-sockets-vs-tcp-ports)


