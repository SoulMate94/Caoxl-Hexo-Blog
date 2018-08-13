---
title: Linux 命令 「nmap」
date: 2018-07-11 10:47:41
categories: Linux cmd
tags: [Linux]
---

> **nmap命令**是一款开放源代码的网络探测和安全审核工具，它的设计目标是快速地扫描大型网络。

<!-- more -->

# 安装`nmap`

```
    yum install nmap
```

# 语法

```
    nmap (选项) (参数)
```

# 选项

- `-O`:   激活操作探测
- `-P0`:  值进行扫描, 不`ping`主机
- `-PT`:  是同`TCP`的`ping`
- `-sV`:  探测服务版本信息
- `-sP`:  ping扫描, 仅发现目标主机是否存活
- `-ps`:  发送同步(SYN)报文
- `-PU`:  发送udp ping
- `-PE`:  强制执行直接的`TCMPping`
- `-PB`:  默认模式, 可以使用`ICMPping`和`TCPping`
- `-6`:   使用`IPv6`地址
- `-v`:   得到更多选项信息
- `-d`:   增加调试信息的输出
- `-oN`:  以人民可阅读的格式输出
- `-oX`:  以xml格式向指定文件输出信息
- `-oM`:  以机器可阅读的格式输出
- `-A`:   使用所有高级扫描选项
- `--resume`: 继续上次执行完的扫描
- `-P`:   指定要扫描的端口，可以是一个单独的端口，用逗号隔开多个端口，使用`“-”`表示端口范围
- `-e`:   在多网络接口`Linux`系统中, 指定扫描使用的网络接口
- `-g`:   将指定的端口作为源端口进行扫描
- `--ttl`: 指定发送的扫描报文进行扫描
- `--packet-trace`: 显示扫描过程中收发报文统计
- `--scanflags`: 设置在扫描报文中的`TCP`标志

# 参数

- `ip`地址: 指定待扫描报文中的`TCP`地址

# 实例

- 使用`nmap`扫描`www.caoxl.com`的开放端口

## 使用主机名扫描

```
    [root@izj6c6djex81rijczh0t8yz ~]# nmap www.caoxl.com
    
    Starting Nmap 6.40 ( http://nmap.org ) at 2018-07-11 10:47 CST
    Nmap scan report for www.caoxl.com (47.91.221.85)
    Host is up (0.00030s latency).
    Not shown: 996 filtered ports
    PORT     STATE  SERVICE
    22/tcp   open   ssh
    80/tcp   open   http
    443/tcp  open   https
    3389/tcp closed ms-wbt-server
    
    Nmap done: 1 IP address (1 host up) scanned in 4.69 seconds
```

## 使用IP地址扫描

```
    [root@izj6c6djex81rijczh0t8yz ~]# nmap 106.15.38.11
    
    Starting Nmap 6.40 ( http://nmap.org ) at 2018-07-11 11:10 CST
    Nmap scan report for 106.15.38.11
    Host is up (0.035s latency).
    Not shown: 985 closed ports
    PORT     STATE    SERVICE
    22/tcp   open     ssh
    25/tcp   filtered smtp
    42/tcp   filtered nameserver
    80/tcp   open     http
    135/tcp  filtered msrpc
    139/tcp  filtered netbios-ssn
    443/tcp  open     https
    445/tcp  filtered microsoft-ds
    593/tcp  filtered http-rpc-epmap
    1025/tcp filtered NFS-or-IIS
    1068/tcp filtered instl_bootc
    1434/tcp filtered ms-sql-m
    3128/tcp filtered squid-http
    3306/tcp open     mysql
    4444/tcp filtered krb524
    
    Nmap done: 1 IP address (1 host up) scanned in 1.77 seconds
```

## 扫描使用`“-v”`选项

使用“ –v “选项后给出了远程机器更详细的信息

```
    [root@izj6c6djex81rijczh0t8yz ~]# nmap -v caoxl.com
    
    Starting Nmap 6.40 ( http://nmap.org ) at 2018-07-11 11:10 CST
    Initiating Ping Scan at 11:10
    Scanning caoxl.com (47.91.221.85) [4 ports]
    Completed Ping Scan at 11:10, 0.00s elapsed (1 total hosts)
    Initiating Parallel DNS resolution of 1 host. at 11:10
    Completed Parallel DNS resolution of 1 host. at 11:10, 0.03s elapsed
    Initiating SYN Stealth Scan at 11:10
    Scanning caoxl.com (47.91.221.85) [1000 ports]
    Discovered open port 80/tcp on 47.91.221.85
    Discovered open port 22/tcp on 47.91.221.85
    Discovered open port 443/tcp on 47.91.221.85
    Completed SYN Stealth Scan at 11:11, 4.22s elapsed (1000 total ports)
    Nmap scan report for caoxl.com (47.91.221.85)
    Host is up (0.00026s latency).
    Not shown: 996 filtered ports
    PORT     STATE  SERVICE
    22/tcp   open   ssh
    80/tcp   open   http
    443/tcp  open   https
    3389/tcp closed ms-wbt-server
    
    Read data files from: /usr/bin/../share/nmap
    Nmap done: 1 IP address (1 host up) scanned in 4.30 seconds
               Raw packets sent: 2002 (88.064KB) | Rcvd: 20 (816B)
```

## 扫描整个子网

```
    [root@izj6c6djex81rijczh0t8yz ~]# nmap 47.91.221.*
    
    Starting Nmap 6.40 ( http://nmap.org ) at 2018-07-11 11:12 CST
    Nmap scan report for 47.91.221.0
    Host is up (0.00034s latency).
    Not shown: 999 filtered ports
    PORT   STATE  SERVICE
    80/tcp closed http
    
    Nmap scan report for 47.91.221.1
    Host is up (0.00087s latency).
    Not shown: 996 filtered ports
    PORT     STATE  SERVICE
    22/tcp   open   ssh
    80/tcp   open   http
    443/tcp  open   https
    3389/tcp closed ms-wbt-server
    
    Nmap scan report for 47.91.221.2
    Host is up (0.00084s latency).
    Not shown: 982 closed ports
    PORT     STATE    SERVICE
    25/tcp   filtered smtp
    135/tcp  open     msrpc
    139/tcp  open     netbios-ssn
    445/tcp  open     microsoft-ds
    666/tcp  open     doom
    6666/tcp open     irc
    8000/tcp open     http-alt
    8001/tcp open     vcom-tunnel
    8002/tcp open     teradataordbms
    8007/tcp open     ajp12
    8008/tcp open     http
    8009/tcp open     ajp13
    8100/tcp open     xprint-server
    9000/tcp open     cslistener
    9001/tcp open     tor-orport
    9002/tcp open     dynamid
    9003/tcp open     unknown
    9009/tcp open     pichat
    
    ...
```

# 参考

- [给Linux系统/网络管理员准备的Nmap命令的29个实用范例](http://blog.jobbole.com/54595/)