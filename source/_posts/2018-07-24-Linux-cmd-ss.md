---
title: Linux 命令 「ss」
date: 2018-07-24 15:23:36
categories: Linux cmd
tags: Linux
---

> **ss命令**用来显示处于活动状态的套接字信息。`ss`命令可以用来获取`socket`统计信息，它可以显示和`netstat`类似的内容。

<!-- more -->

天下武功唯快不破。`ss`快的秘诀在于，它利用到了TCP协议栈中`tcp_diag`。`tcp_diag`是一个用于分析统计的模块，可以获得`Linux` 内核中第一手的信息，这就确保了`ss`的快捷高效。当然，如果你的系统中没有`tcp_diag`, `ss`也可以正常运行，只是效率会变得稍慢。

> TCP用主机的IP地址加上主机上的端口号作为TCP连接的端点，这种端点就叫做套接字(socket)或插口。

# 语法

```
    ss (选项)
```

# 选项

- `-h`: 显示帮助信息
- `-V`: 显示指令版本信息
- `-n`: 不解析服务名称, 以数字方式显示
- `-a`: 显示所有的套接字
- `-l`: 显示处于监听状态的套接字
- `-o`: 显示计时器信息
- `-m`: 显示套接字的内存使用情况
- `-p`: 显示使用套接字的进程信息
- `-i`: 显示内部的`TCP`信息
- `-4`: 只显示`ipv4`的套接字
- `-6`: 只显示`ipv6`的套接字
- `-t`: 只显示`tcp`套接字
- `-u`: 只显示`udp`套接字
- `-d`: 只显示`DCCP`套接字
- `-w`: 仅显示`RAW`套接字
- `-x`: 仅显示`UNIX`域套接字

# 实例

## 显示`ICP`连接

```
    [root@caoxianliang ~]# ss -t -a
    State       Recv-Q Send-Q               Local Address:Port                                Peer Address:Port                
    LISTEN      0      128                              *:https                                          *:*                    
    LISTEN      0      128                      127.0.0.1:cslistener                                     *:*                    
    LISTEN      0      50                               *:mysql                                          *:*                    
    LISTEN      0      128                              *:http                                           *:*                    
    LISTEN      0      128                              *:ssh                                            *:*                    
    ESTAB       0      0                     172.31.45.94:43842                              106.11.248.51:http                 
    ESTAB       0      232                   172.31.45.94:ssh                                 59.41.94.230:44067                
    LISTEN      0      128                             :::smc-https                                     :::*  
```


## 显示`Sockets`摘要

```
    [root@caoxianliang ~]# ss -s
    Total: 107 (kernel 192)
    TCP:   9 (estab 2, closed 1, orphaned 0, synrecv 0, timewait 0/0), ports 0
    
    Transport Total     IP        IPv6
    *	  192       -         -        
    RAW	  0         0         0        
    UDP	  6         4         2        
    TCP	  8         7         1        
    INET	  14        11        3        
    FRAG	  0         0         0   
```

## 列出所有打开的网络连接端口

```
    [root@caoxianliang ~]# ss -l
    Netid State      Recv-Q Send-Q             Local Address:Port                              Peer Address:Port                
    nl    UNCONN     0      0                           rtnl:ntpd/461                                      *                     
    nl    UNCONN     0      0                           rtnl:kernel                                        *                     
    nl    UNCONN     0      0                           rtnl:ntpd/461                                      *                     
    nl    UNCONN     4352   0                        tcpdiag:ss/4090                                       *                     
    nl    UNCONN     768    0                        tcpdiag:kernel                                        *                     
    nl    UNCONN     0      0                           xfrm:kernel                                        *                     
    nl    UNCONN     0      0                        selinux:kernel                                        *                     
    nl    UNCONN     0      0                          audit:auditd/433                                    *                     
    nl    UNCONN     0      0                          audit:kernel                                        * 
    ...
```

## 查看进程使用的`Sockets`

```
    [root@caoxianliang ~]# ss -pl
    Netid State      Recv-Q Send-Q             Local Address:Port                              Peer Address:Port                
    nl    UNCONN     0      0                           rtnl:ntpd/461                                      *                     
    nl    UNCONN     0      0                           rtnl:kernel                                        *                     
    nl    UNCONN     0      0                           rtnl:ntpd/461                                      *                     
    nl    UNCONN     4352   0                        tcpdiag:ss/4149                                       *                     
    nl    UNCONN     768    0                        tcpdiag:kernel                                        *                     
    nl    UNCONN     0      0                           xfrm:kernel                                        *                     
    nl    UNCONN     0      0                        selinux:kernel                                        *                     
    nl    UNCONN     0      0                          audit:auditd/433                                    * 
```

## 找到打开套接字/端口应用程序

```
    [root@caoxianliang ~]# ss -pl | grep 80
    u_str  LISTEN     0      50     /tmp/mysql.sock 901802                * 0                     users:(("mysqld",pid=13453,fd=14))
```

## 显示所有的`UDP Sockets`

```
    [root@caoxianliang ~]# ss -u -a
    State       Recv-Q Send-Q               Local Address:Port                                Peer Address:Port                
    UNCONN      0      0                                *:37916                                          *:*                    
    UNCONN      0      0                     172.31.45.94:ntp                                            *:*                    
    UNCONN      0      0                        127.0.0.1:ntp                                            *:*                    
    UNCONN      0      0                                *:ntp                                            *:*                    
    UNCONN      0      0                               :::ntp                                           :::*                    
    UNCONN      0      0                               :::smc-https                                     :::* 
```