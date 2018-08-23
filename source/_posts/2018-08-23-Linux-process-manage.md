---
title: Linux 进程管理工具
date: 2018-08-23 15:31:16
categories: Linux
tags: [Linux, 进程管理]
---

> 任何进程都与文件关联；我们会用到`lsof`工具（`list opened files`），作用是列举系统中已经被打开的文件。在linux环境中，任何事物都是文件，设备是文件，目录是文件，甚至`sockets`也是文件。用好`lsof`命令，对日常的`linux`管理非常有帮助。

<!-- more -->

# 查询进程

- 查询正在运行的进程信息

```
    [root@caoxl ~]# ps -ef
    UID        PID  PPID  C STIME TTY          TIME CMD
    root         1     0  0 Aug03 ?        00:00:14 /usr/lib/systemd/systemd --switched-root --sys
    root         2     0  0 Aug03 ?        00:00:00 [kthreadd]
    root         3     2  0 Aug03 ?        00:00:06 [ksoftirqd/0]
```

- 查询归属于用户caoxl的进程

```
    [root@caoxl ~]# ps -ef | grep caoxl
    root      4243  3740  0 15:36 pts/0    00:00:00 grep --color=auto caoxl

    [root@caoxl ~]# ps -lu caoxl
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    [root@caoxl ~]# ps -lu git
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
```

- 查询进程ID（适合只记得部分进程字段）

```
    pgrep 查找进程
    
    # 查询进程名中含有re的进程
    [root@caoxl ~]# pgrep -l re
    2 kthreadd
```

- 以完整的格式显示所有的进程

```
    [root@caoxl ~]# ps -ajx
     PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
        0     1     1     1 ?           -1 Ss       0   0:14 /usr/lib/systemd/systemd --switched-r
        0     2     0     0 ?           -1 S        0   0:00 [kthreadd]
        2     3     0     0 ?           -1 S        0   0:06 [ksoftirqd/0]
```

- 显示进程信息，并实时更新

```
    [root@caoxl ~]# top
    top - 15:39:34 up 20 days,  6:00,  1 user,  load average: 0.00, 0.01, 0.05
    Tasks:  91 total,   1 running,  90 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem :  1015436 total,   103440 free,   309648 used,   602348 buff/cache
    KiB Swap:        0 total,        0 free,        0 used.   527000 avail Mem 
    
      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                   
        1 root      20   0   43392   3080   1888 S  0.0  0.3   0:14.48 systemd                   
        2 root      20   0       0      0      0 S  0.0  0.0   0:00.17 kthreadd                  
        3 root      20   0       0      0      0 S  0.0  0.0   0:06.59 ksoftirqd/0 
```

- 查看端口占用的进程状态：

```
    [root@caoxl ~]# lsof -i:6789
    COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    python  1448 root    4u  IPv6  15329      0t0  TCP *:smc-https (LISTEN)
    python  1448 root    6u  IPv6  15330      0t0  UDP *:smc-https 
    python  1448 root   10u  IPv6 774840      0t0  TCP caoxl:smc-https->61.140.74.96:25124 (ESTABLISHED)
```

- 查看用户username的进程所打开的文件

```
    lsof -u username
```

- 查询mysqld进程当前打开的文件

```
    [root@caoxl ~]# lsof -c mysqld
    COMMAND    PID  USER   FD   TYPE             DEVICE  SIZE/OFF    NODE NAME
    mysqld_sa  777  root  cwd    DIR              253,1      4096 1445566 /usr/local/mysql
    mysqld_sa  777  root  rtd    DIR              253,1      4096       2 /
    mysqld_sa  777  root  txt    REG              253,1    964544 1055537 /usr/bin/bash
    ...
```

- 查询指定的进程ID(777)打开的文件：

```
    [root@caoxl ~]# lsof -p 777
    COMMAND   PID USER   FD   TYPE             DEVICE  SIZE/OFF    NODE NAME
    mysqld_sa 777 root  cwd    DIR              253,1      4096 1445566 /usr/local/mysql
    mysqld_sa 777 root  rtd    DIR              253,1      4096       2 /
    mysqld_sa 777 root  txt    REG              253,1    964544 1055537 /usr/bin/bash
    mysqld_sa 777 root  mem    REG              253,1     62184 1053860 /usr/lib64/libnss_files-2.17.so
    ...
```

- 查询指定目录下被进程开启的文件（使用+D 递归目录）：

```
    [root@caoxl ~]# lsof +d /usr/local
    COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
    mysqld_sa 777 root  cwd    DIR  253,1     4096 1445566 /usr/local/mysql
```

# 终止进程

- 杀死指定PID的进程 (PID为Process ID)

```
    kill PID
```

- 杀死相关进程

```
    kill -p 1234
```

- 杀死job工作 (job为job number)

```
    kill $job
```

# 进程监控

- 查看系统中使用CPU、使用内存最多的进程；

```
    top
```

输入top命令后，进入到交互界面；接着输入字符命令后显示相应的进程状态：
对于进程，平时我们最常想知道的就是哪些进程占用CPU最多，占用内存最多。以下两个命令就可以满足要求:

- `P`: 根据CPU使用百分比大小进行排序。
- `M`: 根据驻留内存大小进行排序
- `i`: 使top不显示任何闲置或者僵死进程。


# 分析线程栈

使用命令`pmap`，来输出进程内存的状况，可以用来分析线程堆栈；

```
    [root@caoxl ~]# pmap 777
    777:   /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/var --pid-file=/usr/local/mysql/var/izj6c6djex81rijczh0t8yz.pid
    0000000000400000    884K r-x-- bash
    00000000006dd000      4K r---- bash
    00000000006de000     36K rw--- bash
    ...
```

# 综合运用

- 将用户`caoxl`下的所有进程名以av_开头的进程终止:

```
    ps -u caoxl |  awk '/av_/ {print "kill -9 " $1}' | sh
```

- 将用户`caoxl`下所有进程名中包含HOST的进程终止:

```
    ps -fe| grep caoxl|grep HOST |awk '{print $2}' | xargs kill -9;
```