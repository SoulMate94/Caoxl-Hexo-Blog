---
title: Linux 命令 「iotop」
date: 2018-06-25 09:47:31
categories: Linux cmd
tags: [Linux]
---

> `iotop`命令是一个用来监视磁盘`I/O`使用状况的`top`类工具。
`iotop`具有与`top`相似的`UI`，其中包括`PID`、用户、`I/O`、进程等相关信息。

<!-- more -->

# 安装

## Ubuntu

```
    apt-get install iotop
```

## Centos

```
    yum install iotop
```

## 编译安装

```
    wget http://guichaz.free.fr/iotop/files/iotop-0.4.4.tar.gz    
    tar zxf iotop-0.4.4.tar.gz    
    python setup.py build    
    python setup.py install    
```

# 命令/参数

- `-o`:   只显示有io操作的进程
- `-b`:   批量显示,无交互,主要用作记录到文件
- `-n NUM`:   显示NUM次,主要用于非交互模式
- `-d SEC`:   间隔SEC秒显示一次
- `-p PID`:   监控的进程pid
- `-u USER`:  监控的进程用户

## iotop常用快捷键:

1. 左右箭头：改变排序方式，默认是按`IO`排序。
2. `r`：改变排序顺序。
3. `o`：只显示有IO输出的进程。
4. `p`：进程/线程的显示方式的切换。
5. `a`：显示累积使用量。
6. `q`：退出。

## 实例

```
    Total DISK READ :	0.00 B/s | Total DISK WRITE :       0.00 B/s
    Actual DISK READ:	0.00 B/s | Actual DISK WRITE:       0.00 B/s
      TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND                                        
        1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % systemd --system --deserialize 21
        2 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthreadd]
        3 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [ksoftirqd/0]
    10826 be/4 mysql       0.00 B/s    0.00 B/s  0.00 %  0.00 % mysqld --basedir=/usr/lo~mp/mysql.sock --port=3306
        5 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kworker/0:0H]
        7 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/0]
        8 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_bh]
        9 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [rcu_sched]
       10 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [watchdog/0]
       12 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kdevtmpfs]
       13 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [netns]
       14 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [khungtaskd]
       15 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [writeback]
       16 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kintegrityd]
       17 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [bioset]
       18 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kblockd]
       19 be/0 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [md]
       25 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kswapd0]
```