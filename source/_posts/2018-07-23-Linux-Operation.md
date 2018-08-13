---
title: Linux 「运维命令」
date: 2018-07-23 09:27:41
categories: Linux
tags: [Linux]
---

> Linux 运维命令总结

<!-- more -->

# 查看某个端口进程

- `netstat`

```
    [root@caoxianliang ~]# netstat -tnlp | grep 80
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      17121/nginx: master 
```

- `lsof`

```
    [root@caoxianliang ~]# lsof -i:80
    COMMAND     PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    python     1175 root   17u  IPv4 930540      0t0  TCP caoxianliang:37217->36.110.220.15:http (ESTABLISHED)
    AliYunDun 17050 root   19u  IPv4 926114      0t0  TCP caoxianliang:58918->106.11.68.13:http (ESTABLISHED)
    nginx     17121 root    8u  IPv4 926506      0t0  TCP *:http (LISTEN)
    nginx     17122  www    8u  IPv4 926506      0t0  TCP *:http (LISTEN)
```

# 删除`0`字节文件

```
    find -type f -size 0 -exec rm -rf {} \;
```

# 查看进程

- 按内存从大到小排列

```
    ps -e -o "%C : %p : %z : %a" | sort -k5 -nr
```

# 按`CPU`利用率从大到小排列

```
    ps -e -o "%C : %p : %z : %a" | sort -nr
```

# 查看`HTTP`的并发请求数及其`TCP`连接状态

```
    netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

# 如何杀掉`MYSQL`进程

```
    ps aux |grep mysql |grep -v grep  |awk '{print $2}' |xargs kill -9
```

```
    killall -TERM mysqld
```

- 查看`mysql`进程

```
    service mysql status
```

# 显示运行3级别开启的服务

```
    ls /etc/rc3.d/S* |cut -c 15-
```

# 查看内存的大小

```
    free -m |grep "Mem" | awk '{print $2}'
```

# 统计一下服务器下面所有的`jpg`的文件的大小

```
    find / -name *.jpg -exec wc -c {} \;|awk '{print $1}'|awk '{a+=$1}END{print a}'
```

# 查看`CPU` 负载

- 1. `cat /proc/loadavg`

```
    [root@caoxianliang ~]# cat /proc/loadavg
    0.00 0.01 0.05 3/123 13560
```

检查前三个输出值是否超过了系统逻辑CPU的4倍。 

- 2. `mpstat 1 1`

```
    [root@caoxianliang ~]# mpstat 1 1
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    10:07:25 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
    10:07:26 AM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
    Average:     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
```

检查`%idle`是否过低(比如小于5%)


# 找出占用空间最多的文件或目录

```
    du -cks * | sort -rn | head -n 10
```

```
    [root@caoxianliang ~]# du -cks * | sort -rn | head -n 10
    669900	total
    348556	bin
    159544	lib
    124740	mysql-test
    30000	var
    2800	sql-bench
    2644	share
    732	man
    708	include
    100	support-files
```

# 查看磁盘`I/O`负载

```
    iostat -x 1 2
```

```
    [root@caoxianliang ~]# iostat -x 1 2
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.20    0.01    0.11    0.02    0.00   99.65
    
    Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    vda               0.00     0.34    0.09    0.77     1.49     5.37    15.97     0.00    5.54    7.19    5.35   0.33   0.03
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.00    0.00    0.00    0.00    0.00  100.00
    
    Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    vda               0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
    
```

> 检查`I/O`使用率(`%util`)是否超过100%

# 查看网络负载

```
    sar -n DEV
```

```
    [root@caoxianliang ~]# sar -n DEV
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    12:00:01 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
    12:10:01 AM      eth0      0.14      0.14      0.01      0.04      0.00      0.00      0.00
    12:10:01 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
    12:20:01 AM      eth0      0.08      0.10      0.00      0.01      0.00      0.00      0.00
    12:20:01 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
    12:30:01 AM      eth0      0.29      0.23      0.02      0.03      0.00      0.00      0.00
    12:30:01 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
    12:40:01 AM      eth0      0.26      0.17      0.02      0.02      0.00      0.00      0.00
```

> 检查网络流量(`rxbyt/s`, `txbyt/s`)是否过高

# 查看网络错误

```
    [root@caoxianliang ~]# netstat -i
    Kernel Interface table
    Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
    eth0      1500  1648927      0      0 0       1454542      0      0      0 BMRU
    lo       65536      306      0      0 0           306      0      0      0 LRU
```

# 查看网络连接数目

```
    [root@caoxianliang ~]# netstat -an | grep -E "^(tcp)" | cut -c 68- | sort | uniq -c | sort -n
          1  FIN_WAIT1  
          1  LAST_ACK   
          5  TIME_WAIT  
          6  ESTABLISHED
          6  LISTEN  
```

# 查看进程总数

```
    [root@caoxianliang ~]# ps aux | wc -l
    74
```

> 检查进程个数是否正常 (比如超过250)

# 查看可运行进程数目

```
    [root@caoxianliang ~]# vmstat 1 5
    procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     3  0      0 108564 135436 614224    0    0     2     5   41    7  0  0 100  0  0
     0  0      0 108564 135436 614224    0    0     0     0  118  179  1  0 99  0  0
     0  0      0 108564 135436 614224    0    0     0     0  128  189  0  0 100  0  0
     0  0      0 108564 135436 614224    0    0     0     0  138  208  0  0 100  0  0
     0  0      0 108564 135436 614224    0    0     0     0  128  187  0  1 99  0  0
```

> 列给出的是可运行进程的数目，检查其是否超过系统逻辑CPU的4倍

# 观察是否有异常进程出现

```
    top -id 1
```

# 检查登录用户是否过多

- `uptime`

```
    [root@caoxianliang ~]# uptime
     11:01:15 up 9 days, 17:35,  1 user,  load average: 0.14, 0.14, 0.10
```

- `who |wc -l`

```
    [root@caoxianliang ~]# who | wc -l
    1
```

# 查看系统时间

- `date`

```
    [root@caoxianliang ~]# date
    Mon Jul 23 11:03:45 CST 2018
    
    [root@caoxianliang ~]# date --date=yesterday
    Sun Jul 22 11:34:21 CST 2018
```

- `cal`

```
    [root@caoxianliang ~]# cal
          July 2018     
    Su Mo Tu We Th Fr Sa
     1  2  3  4  5  6  7
     8  9 10 11 12 13 14
    15 16 17 18 19 20 21
    22 23 24 25 26 27 28
    29 30 31
```

# 检查打开文件总数是否过多

- `lsof |wc -l`

```
    [root@caoxianliang ~]# lsof | wc -l
    3957
```

# 杀掉80端口相关的进程

```
    lsof -i :80|grep -v "ID"|awk '{print "kill -9",$2}'|sh
```

> 使用 `netstat -lntp` 查看


# `tcpdump` 抓包 ，用来防止80端口被人攻击时可以分析数据

```
    tcpdump -c 10000 -i eth0 -n dst port 80 > /root/packets
```

**实例:**

```
    [root@caoxianliang ~]# tcpdump -c 100 -i eth0 -n dst port 80 > /root/packets
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
    100 packets captured
    101 packets received by filter
    0 packets dropped by kernel
```

**然后检查`IP`的重复数 并从小到大排序 注意 `"-t\ +0"`   中间是两个空格**

```
    less packets | awk {'printf $3"\n"'} | cut -d. -f 1-4 | sort | uniq -c | awk {'printf $1" "$2"\n"'} | sort -n -t\ +0
```

**实例:**

```
    [root@caoxianliang ~]# less packets | awk {'printf $3"\n"'} | cut -d. -f 1-4 | sort | uniq -c | awk {'printf $1" "$2"\n"'} | sort -n 
    100 172.31.45.94
```

# 查看有多少个活动的`php-cgi`进程

```
    netstat -anp | grep php-cgi | grep ^tcp | wc -l
```

**实例:**

```
    [root@caoxianliang ~]# netstat -anp | grep php-cgi | grep ^tcp | wc -l
    0
```

# 查看系统自启动的服务

```
    chkconfig --list | awk '{if ($5 == "3:on") print $1}'
```

**实例:**

```
     [root@caoxianliang ~]# chkconfig --list | awk '{if ($5=="3:on") print $1}'
     
     Note: This output shows SysV services only and does not include native
           systemd services. SysV configuration data might be overridden by native
           systemd configuration.
     
           If you want to list systemd services use 'systemctl list-unit-files'.
           To see services enabled on particular target use
           'systemctl list-dependencies [target]'.
     
     aegis
     agentwatch
     mysql
     network
     nginx
     php-fpm
     shadowsocks
```

# 查看硬件制作商

```
    dmidecode -s system-product-name
```

**实例:**

```
    [root@caoxianliang ~]# dmidecode -s system-product-name
    Alibaba Cloud ECS
```

# 显示最常用的20条命令

```
    cat ~/.bash_history |grep -v ^# |awk '{print $1}' |sort |uniq -c |sort -nr |head -20
```

**实例:**

```
    [root@caoxianliang ~]# cat .bash_history |grep -v ^# |awk '{print $1}' |sort |uniq -c |sort -nr |head -20
        265 ls
        200 cd
         43 vim
         28 rm
         28 ll
         28 cat
         24 rz
         24 ps
         15 yum
         15 uname
         14 rpm
         12 mkdir
         11 rar
         10 sh
         10 php
          9 whereis
          9 df
          9 chage
          8 tar
          8 curl
```

