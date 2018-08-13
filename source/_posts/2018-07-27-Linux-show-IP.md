---
title: Linux终端查看服务器IP
date: 2018-07-27 14:59:01
categories: Linux
tags: [Linux, IP]
---

> IP地址是计算机网络中用于区分不同主机的数字标识，所有连接到网络中的设备均采用IP协议来进行通信。IP地址的两个主要功能就是区分不同的网络或主机以及用于定位网络地址。

<!-- more -->

# 查看服务器公网IP

## 使用`dig`工具

- 安装`dig`命令

```
    yum install bind-utils
```

> `dig` (`domain information groper`)是一个用于查询`DNS`服务器的工具，为了查询主机的公网IP地址，可以通过查询`opendns.com`的反向解析结果来得到。
> > `dig +short myip.opendns.com @resolver1.opendns.com`

```
    [root@caoxianliang test]# dig +short myip.opendns.com @resolver1.opendns.com
    47.91.221.85
```

## 使用`host`命令

> `host`命令是一条简便的DNS查询工具，同样可以通过向`opendns.com`发送查询请求来得到本机的公网地址

```
    [root@caoxianliang test]# host myip.opendns.com resolver1.opendns.com | grep "myip.opendns.com has" | awk '{print $4}'
    47.91.221.85
```

## 使用`wget`下载工具

> `wget`是一种支持 `HTTP`, `HTTPS`, `FTP`等多种协议的命令行下载工具，于是我们可以通过`wget`第三方网站来得到本机的`IP`地址

```
    [root@caoxianliang test]# wget -qO- http://ipecho.net/plain | xargs echo
    47.91.221.85

    [root@caoxianliang test]# wget -qO - icanhazip.com
    47.91.221.85
```

> 两条命令的执行结果是一致的，只是访问的第三方网站不相同。

## 使用`cURL`命令行下载工具

> 既然`wget`可以用，那么功能更加强大的`curl`工具当然也没有问题。`curl`是一种支持多种网络协议（`HTTP`、`HTTPS`、`FILE`、`FTP`、`FTPS`等）的上传下载工具，同样可以访问第三方网站查询服务器的公网IP地址

```
    [root@caoxianliang test]# curl ifconfig.co
    47.91.221.85

    [root@caoxianliang test]# curl ifconfig.me
    47.91.221.85

    [root@caoxianliang test]# curl icanhazip.com
    47.91.221.85
```

# 查看服务器内网`IP`


## 使用 `ifconfig`

```
    [root@caoxianliang test]# ifconfig
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 172.31.45.94  netmask 255.255.240.0  broadcast 172.31.47.255
            ether 00:16:3e:01:a4:4f  txqueuelen 1000  (Ethernet)
            RX packets 3714175  bytes 2541275565 (2.3 GiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 2733364  bytes 2746165972 (2.5 GiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 3046  bytes 3781560 (3.6 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 3046  bytes 3781560 (3.6 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## 使用 `ip addr`

```
    [root@caoxianliang test]# ip addr 
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:16:3e:01:a4:4f brd ff:ff:ff:ff:ff:ff
        inet 172.31.45.94/20 brd 172.31.47.255 scope global eth0
           valid_lft forever preferred_lft forever
```

# 参考

- [Linux终端查看服务器公网IP地址的四条命令](https://www.daehub.com/archives/3630.html)