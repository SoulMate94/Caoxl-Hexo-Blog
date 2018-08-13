---
title: Linux 面试题集
date: 2018-07-13 14:59:58
categories: Linux
tags: [Linux]
---

> 多说无益, 实战为王

<!-- more -->

# 问:如何查看当前的`Linux`服务器的运行级别？

> 答: `who -r` 和 `runlevel` 命令可以用来查看当前的Linux服务器的运行级别。

- `who -r`

```
    [root@izj6c6djex81rijczh0t8yz ~]# who -r
             run-level 3  2018-06-25 17:02
```

- `runlevel`

```
    [root@izj6c6djex81rijczh0t8yz ~]# runlevel
    N 3
```

# 问: 如何查看Linux的默认网关？

> 答: 用 `route -n` 和 `netstat -nr` 命令,我们可以查看默认网关。除了默认的网关信息,这两个命令还可以显示当前的路由表。

- `route -n`

```
    [root@izj6c6djex81rijczh0t8yz ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         172.31.47.253   0.0.0.0         UG    0      0        0 eth0
    169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
    172.31.32.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
```

- `netstat -nr`

```
    [root@izj6c6djex81rijczh0t8yz ~]# netstat -nr
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    0.0.0.0         172.31.47.253   0.0.0.0         UG        0 0          0 eth0
    169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
    172.31.32.0     0.0.0.0         255.255.240.0   U         0 0          0 eth0
```

# 问: 如何在Linux上重建初始化内存盘镜像文件？

> 答: 在CentOS 5.X / RHEL 5.X中,可以用`mkinitrd`命令来创建初始化内存盘文件,举例如下

```
    [root@izj6c6djex81rijczh0t8yz ~]# mkinitrd -f -v /boot/initrd-$(uname -r).img $(uname -r)
```

如果你想要给特定的内核版本创建初始化内存盘,你就用所需的内核名替换掉 `uname -r`

> 在CentOS 6.X / RHEL 6.X中,则用`dracut`命令来创建初始化内存盘文件,举例如下:

```
    [root@izj6c6djex81rijczh0t8yz ~]# dracut -f
```

# 问: `cpio`命令是什么？

> 答: `cpio`就是复制入和复制出的意思。cpio可以向一个归档文件（或单个文件）复制文件、列表,还可以从中提取文件。

# 问: `patch`命令是什么？如何使用？

> 答: 顾名思义,`patch`命令就是用来将修改（或补丁）写进文本文件里。`patch`命令通常是接收`diff`的输出并把文件的旧版本转换为新版本

# 问: `aspell`有什么用 ?

> 答: 顾名思义,`aspell`就是`Linux`操作系统上的一款交互式拼写检查器。aspell命令继任了更早的一个名为ispell的程序,并且作为一款免费替代品 ,最重要的是它非常好用

# 问: 如何从命令行查看域`SPF`记录 ？

> 答: 我们可以用`dig`命令来查看域`SPF`记录。举例如下:

```
    [root@izj6c6djex81rijczh0t8yz test]# dig -t TXT google.com
    
    ; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> -t TXT google.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21277
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
    
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 4096
    ;; QUESTION SECTION:
    ;google.com.			IN	TXT
    
    ;; ANSWER SECTION:
    google.com.		300	IN	TXT	"docusign=05958488-4752-4ef2-95eb-aa7ba8a3bd0e"
    google.com.		300	IN	TXT	"facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
    google.com.		300	IN	TXT	"v=spf1 include:_spf.google.com ~all"
    
    ;; Query time: 13 msec
    ;; SERVER: 100.100.2.136#53(100.100.2.136)
    ;; WHEN: Fri Jul 13 15:17:02 CST 2018
    ;; MSG SIZE  rcvd: 217
```

# 问: 如何识别`Linux`系统中指定文件(`/etc/fstab`)的关联包？

> rpm -qf /etc/fstab

```
    [root@izj6c6djex81rijczh0t8yz test]# rpm -qf /etc/fstab
    setup-2.8.71-9.el7.noarch
```

以上命令能列出提供`“/etc/fstab”`这个文件的包。


# 问: 哪条命令用来查看`bond0`的状态？

```
    cat /proc/net/bonding/bond0
```

# 问: `Linux`系统中的`/proc`文件系统有什么用 ？

> 答: `/proc`文件系统是一个基于内存的文件系统,其维护着关于当前正在运行的内核状态信息,其中包括`CPU`、内存、分区划分、I/O地址、直接内存访问通道和正在运行的进程。这个文件系统所代表的并不是各种实际存储信息的文件,它们指向的是内存里的信息。`/proc`文件系统是由系统自动维护的。

# 问: 如何在`/usr`目录下找出大小超过10MB的文件？

> find /usr -size +10M


# 问: 如何在`/home`目录下找出`120`天之前被修改过的文件？

> find /home -mtime +120


# 问: 如何在`/var`目录下找出`90`天之内未被访问过的文件？

>  find /var \! -atime -90

# 问: 在整个目录树下查找文件`“core”`,如发现则无需提示直接删除它们。

> find / -name core -exec rm {} \;

上个这个命令不要随意使用!!!

# 问: `strings`命令有什么作用？

> 答: `strings`命令用来提取和显示非文本文件中的文本字符串。

# 问: `tee` 过滤器有什么作用 ?

> 答: `tee` 过滤器用来向多个目标发送输出内容。如果用于管道的话,它可以将输出复制一份到一个文件,并复制另外一份到屏幕上（或一些其它程序）

```
    [root@izj6c6djex81rijczh0t8yz tmp]# cat ll.out 
         1	total 1.5M
         2	-rw-r--r--.  1 root root     18 Oct 15  2017 adjtime
         3	-rw-r--r--   1 root root   1.5K Jun  7  2013 aliases
         4	-rw-r--r--   1 root root    12K Oct 15  2017 aliases.db
         5	drwxr-xr-x.  2 root root   4.0K Jun 13 11:37 alternatives
         6	-rw-------   1 root root    541 Apr 11 09:48 anacrontab
         7	-rw-r--r--   1 root root     55 Apr 11 04:38 asound.conf
         ...
```

在以上例子中,从`ll`输出可以捕获到 `/tmp/ll.out` 文件中,并且同样在屏幕上显示了出来。

# 问: `export PS1 = "$LOGNAME@hostname:\$PWD"`: 这条命令是在做什么？

> 答: 这条`export`命令会更改登录提示符来显示用户名、本机名和当前工作目录。

```
    [root@izj6c6djex81rijczh0t8yz ~]# export PS1="$LOGNAME@hostname:\$PWD"
    root@hostname:/root
```

# 问: `ll | awk '{print $3,"owns",$9}'` 这条命令是在做什么？

> 答: 这条`ll`命令会显示这些文件的文件名和它们的拥有者。

```
    [root@izj6c6djex81rijczh0t8yz ~]# ll | awk '{print $3,"owns",$9}'
     owns 
    root owns curl
    root owns dnf-0.6.4-2.sdl7.noarch.rpm
    root owns dnf-conf-0.6.4-2.sdl7.noarch.rpm
    root owns letsencrypt
    root owns linux
    root owns lnmp-install.log
    mysql owns package.xml
    root owns python-dnf-0.6.4-2.sdl7.noarch.rpm
    root owns redis-4.0.2
    root owns test
```

# 问: `Linux`中的`at`命令有什么用 ？

> 答: `at`命令用来安排一个程序在未来的做一次一次性执行。所有提交的任务都被放在 `/var/spool/at` 目录下并且到了执行时间的时候通过`atd`守护进程来执行

# 问: `Linux`中`lspci`命令的作用是什么？

> 答: `lspci`命令用来显示你的系统上`PCI`总线和附加设备的信息。指定`-v`,`-vv`或`-vvv`来获取越来越详细的输出,加上`-r`参数的话,命令的输出则会更具有易读性。

# 问: 如何看当前`Linux`系统有几颗物理`CPU`和每颗`CPU`的核数？

```
    [root@izj6c6djex81rijczh0t8yz ~]# cat /proc/cpuinfo|grep -c 'physical id'
    1
    [root@izj6c6djex81rijczh0t8yz ~]# cat /proc/cpuinfo|grep -c 'processor'
    1
```

# 问: 查看系统负载有两个常用的命令,是哪两个？这三个数值表示什么含义呢？

```
    [root@shuidianbang ~]# w
     16:28:08 up 319 days, 58 min,  1 user,  load average: 0.00, 0.00, 0.00
    USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
    root     pts/0    59.41.94.109     14:52    0.00s  0.10s  0.00s w
    
    
    [root@shuidianbang ~]# uptime
     16:28:13 up 319 days, 58 min,  1 user,  load average: 0.00, 0.00, 0.00
```

其中`load average`即**系统负载**,三个数值分别表示`一分钟`、`五分钟`、`十五分钟`内系统的平均负载,即平均任务数。

# 问: `vmstat` 命令 `r`, `b`, `si`, `so`, `bi`, `bo` 这几列表示什么含义呢？

```
    [root@shuidianbang ~]# vmstat
    procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     0  0      0 1004452 237088 1612324    0    0     0     6    1    0  0  0 100  0  0	
```

- `r`: `running`,表示正在跑的任务数
- `b`: `blocked`,表示被阻塞的任务数
- `si`: `si`表示有多少数据从交换分区读入内存
- `so`: 表示有多少数据从内存写入交换分区
- `bi`: 表示有多少数据从磁盘读入内存
- `bo`: 表示有多少数据从内存写入磁盘
- `i`: `input`, 进入内存
- `o`: `output`, 从内存出去
- `s`: `swap`, 交换分区
- `b`: `block`, 块设备, 磁盘

**单位都是KB**

# 问: `Linux`系统里,您知道`buffer`和`cache`如何区分吗？

> 答: `buffer`和`cache`都是内存中的一块区域,
> 当CPU需要写数据到磁盘时,由于磁盘速度比较慢,所以`CPU`先把数据存进`buffer`,然后`CPU`去执行其他任务,`buffer`中的数据会定期写入磁盘; 
> 当CPU需要从磁盘读入数据时,由于磁盘速度比较慢,可以把即将用到的数据提前存入`cache`,`CPU`直接从`Cache`中拿数据要快的多

# 问: 使用`top`查看系统资源占用情况时,哪一列表示内存占用呢？

```
    top - 16:54:58 up 319 days,  1:25,  1 user,  load average: 0.00, 0.00, 0.00
    Tasks: 105 total,   1 running, 104 sleeping,   0 stopped,   0 zombie
    Cpu(s):  0.0%us,  0.3%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
    Mem:   4056656k total,  3064012k used,   992644k free,   237088k buffers
    Swap:        0k total,        0k used,        0k free,  1612444k cached
    
    PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
```

- `VIRT`: 虚拟内存用量
- `RES`: 物理内存用量
- `SHR`: 共享内存用量
- `%MEM`: 内存用量

# 问: 如何实时查看网卡流量为多少？如何查看历史网卡流量？

> 答: 安装`sysstat`包,使用`sar`命令查看

```
    yum install -y sysstat
    
    sar -n DEV              # 查看网卡流量, 默认十分钟更新一次
    sar -n DEV 1 10         # 一秒显示一次, 一共显示10次
    sar -n DEV -f /var/log/sa/sa22 # 查看指定日期的流量日志
```

```
    [root@izj6c6djex81rijczh0t8yz ~]# sar
    Linux 3.10.0-862.3.2.el7.x86_64 (izj6c6djex81rijczh0t8yz) 	07/13/2018 	_x86_64_	(1 CPU)
    
    12:00:01 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
    12:10:01 AM     all      0.13      0.00      0.09      0.00      0.00     99.78
    12:20:01 AM     all      0.13      0.00      0.08      0.00      0.00     99.79
    12:30:01 AM     all      0.13      0.00      0.09      0.00      0.00     99.78
    12:40:01 AM     all      0.12      0.03      0.10      0.01      0.00     99.74
    ...
```

# 问: 如何查看当前系统都有哪些进程？

> 答: `ps -aux` 或者`ps -elf`

- `ps -aux`

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps -aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  0.0  0.2  51600  2680 ?        Ss   Jun25   0:47 /usr/lib/systemd/systemd --switch
    root         2  0.0  0.0      0     0 ?        S    Jun25   0:00 [kthreadd]
    root         3  0.0  0.0      0     0 ?        S    Jun25   0:06 [ksoftirqd/0]
    ...
```

- `ps -elf`

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps -elf
    F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
    4 S root         1     0  0  80   0 - 12900 ep_pol Jun25 ?        00:00:47 /usr/lib/systemd/system
    1 S root         2     0  0  80   0 -     0 kthrea Jun25 ?        00:00:00 [kthreadd]
    1 S root         3     2  0  80   0 -     0 smpboo Jun25 ?        00:00:06 [ksoftirqd/0]
    1 S root         5     2  0  60 -20 -     0 worker Jun25 ?        00:00:00 [kworker/0:0H]
    1 S root         7     2  0 -40   - -     0 smpboo Jun25 ?        00:00:00 [migration/0]
```

# 问: `ps` 查看系统进程时,有一列为`STAT`, 如果当前进程的`stat`为`Ss` 表示什么含义？如果为`Z`表示什么含义？

> 答: `S`表示正在休眠; `s`表示主进程; `Z`表示僵尸进程。


# 问: 如何查看系统都开启了哪些端口？

> `netstat -lnp`

```
    [root@izj6c6djex81rijczh0t8yz ~]# netstat -lnp
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      2236/nginx: master  
    tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      2256/php-fpm: maste 
    tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1349/mysqld         
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2236/nginx: master  
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1443/sshd  
```

# 问: 如何查看网络连接状况？

> 答: `netstat -an`

```
    [root@izj6c6djex81rijczh0t8yz ~]# netstat -an
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN     
    tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN     
    tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN     
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
    tcp        0      0 172.31.45.94:41058      140.205.140.205:80      ESTABLISHED
    tcp        0     52 172.31.45.94:22         59.41.94.109:23385      ESTABLISHED
    udp        0      0 172.31.45.94:123        0.0.0.0:*                          
    udp        0      0 127.0.0.1:123           0.0.0.0:*                          
    udp        0      0 0.0.0.0:123             0.0.0.0:*                          
    udp6       0      0 :::123                  :::* 
```

# 问: 想修改`ip`, 需要编辑哪个配置文件, 修改完配置文件后, 如何重启网卡,使配置生效？

> 答: 使用`vi/vim`编辑器编辑网卡配置文件`/etc/sysconfig/network-scripts/ifcft-eth0` (如果是eth1文件名为ifcft-eth1), 内容如下:

```
    DEVICE=eth0
    HWADDR=00:0C:29:06:37:BA
    TYPE=Ethernet
    UUID=0eea1820-1fe8-4a80-a6f0-39b3d314f8da
    ONBOOT=yes
    NM_CONTROLLED=yes
    BOOTPROTO=static
    IPADDR=192.168.147.130
    NETMASK=255.255.255.0
    GATEWAY=192.168.147.2
    DNS1=192.168.147.2
    DNS2=8.8.8.8
```

修改网卡后,可以使用命令重启网卡: 

```
    ifdown eth0
    ifup eth0
```

也可以重启网络服务:

```
    service network restart
```

# 问: 能否给一个网卡配置多个IP? 如果能,怎么配置？

> 答: 可以给一个网卡配置多个IP,配置步骤如下:

```
    cat /etc/sysconfig/network-scripts/ifcfg-eth0#查看eth0的配置
    DEVICE=eth0
    HWADDR=00:0C:29:06:37:BA
    TYPE=Ethernet
    UUID=0eea1820-1fe8-4a80-a6f0-39b3d314f8da
    ONBOOT=yes
    NM_CONTROLLED=yes
    BOOTPROTO=static
    IPADDR=192.168.147.130
    NETMASK=255.255.255.0
    GATEWAY=192.168.147.2
    DNS1=192.168.147.2
    DNS2=8.8.8.8
```

- 1. 新建一个`ifcfg-eth0:1`文件

```
    cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0:1
```

- 2. 修改其内容如下:

```
    vim /etc/sysconfig/network-scripts/ifcfg-eth0:1
    DEVICE=eth0:1
    HWADDR=00:0C:29:06:37:BA
    TYPE=Ethernet
    UUID=0eea1820-1fe8-4a80-a6f0-39b3d314f8da
    ONBOOT=yes
    NM_CONTROLLED=yes
    BOOTPROTO=static
    IPADDR=192.168.147.133
    NETMASK=255.255.255.0
    GATEWAY=192.168.147.2
    DNS1=192.168.147.2
    DNS2=8.8.8.8
```

- 3. 重启网络服务:

```
    service network restart
```

# 问: 如何查看当前主机的主机名,如何修改主机名？要想重启后依旧生效,需要修改哪个配 置文件呢？

- 查看主机名

```
    [root@izj6c6djex81rijczh0t8yz ~]# hostname
    izj6c6djex81rijczh0t8yz
```

- 修改主机名

```
    [root@izj6c6djex81rijczh0t8yz ~]# hostname caoxianliang
    [root@izj6c6djex81rijczh0t8yz ~]# hostname
    caoxianliang
```

- 永久生效需要修改配置文件:

```
    [root@caoxianliang ~]# vim /etc/sysconfig/network
    NETWORKING=yes
    HOSTNAME=caoxianliang
```

# 问: 设置DNS需要修改哪个配置文件？

> (1) 在文件 `/etc/resolv.conf` 中设置`DNS`
> (2) 在文件 `/etc/sysconfig/network-scripts/ifcfg-eth0` 中设置DNS

# 问: 使用`iptables` 写一条规则：把来源IP为`192.168.1.101`访问本机80端口的包直接拒绝

> 答: `iptables -I INPUT -s 192.168.1.101 -p tcp --dport 80 -j REJECT`


# 问: 要想把`iptable`的规则保存到一个文件中如何做？如何恢复？

- 使用`iptables-save`重定向到文件中:

```
    iptables-save > 1.ipt
```

- 使用`iptables-restore`反重定向回来:

```
    iptables-restore < 1.ipt
```

# 问: 如何备份某个用户的任务计划？

> 答: 将`/var/spool/cron/`目录下指定用户的任务计划拷贝到备份目录`cron_bak/`下即可

```
    cp /var/spool/cron/root /tmp/bak/cron_bak/
    
    [root@izj6c6djex81rijczh0t8yz cron]# cat root 
    47 20 * * * curl https://caoxl.com/Aliyun/api_demo/SmsDemo.php
```

# 问: 如何可以把系统中不用的服务关掉？

- 1. 使用可视化工具 `ntsysv`:

```
    yum install ntsysv
```

- 2. 使用命令:

```
    chkconfig service_name off // service_name 指服务名
```

# 问: 如何让某个服务(假如服务名为 `nginx`) 只在3,5两个运行级别开启,其他级别关闭？

- 先关闭所有运行级别:

```
    chkconfig nginx off
```

- 然后打开35运行级别:

```
    chkconfig --level 35 nginx on
```

# 问: `rsync` 同步命令中,下面两种方式有什么不同呢？

```
    (1) rsync -av  /dira/  ip:/dirb/
    (2) rsync -av  /dira/  ip::dirb
```

> 答: (1)前者是通过`ssh`方式同步的
> (2)后者是通过`rsync`服务的方式同步的


# 问: 某个账号登陆`Linux`后,系统会在哪些日志文件中记录相关信息？

> 答: 用户身份验证过程记录在`/var/log/secure`中,登录成功的信息记录在`/var/log/wtmp`

# 问: 在`Linux`系统下如何按照下面要求抓包: 只过滤出访问`http`服务的,目标`ip`为`192.168.0.111`, 一共抓`1000`个包,并且保存到`1.cap`文件中？

> 答: `tcpdump -nn -s0 host 192.168.0.111 and port 80 -c 1000 -w 1.cap`


# 问: `rsync` 同步数据时, 如何过滤出所有`.txt`的文件不同步？

> 答: 加上`--exclude`选项

```
    --exclude="*.txt"
```

# 问: 想在`Linux`命令行下访问某个网站，并且该网站域名还没有解析，如何做 ？

> 答: 在`/etc/hosts`文件中增加一条从该网站域名到其IP的解析记录即可，或者使用`curl -x`


# 问: 自定义解析域名的时候，我们可以编辑哪个文件？是否可以一个`ip`对应多个域名？是否一个域名对应多个`ip`？

> 答: 编辑 `/etc/hosts` ,可以一个`ip`对应多个域名，不可以一个域名对多个`ip`

# 问: 我们可以使用哪个命令查看系统的历史负载（比如说两天前的）？

> 答: `sar -q -f /var/log/sa/sa22`  # 查看22号的系统负载

```
    [root@caoxianliang sa]# sar -q -f /var/log/sa/sa22
    Linux 3.10.0-693.11.1.el7.x86_64 (izj6c6djex81rijczh0t8yz) 	06/22/2018 	_x86_64_	(1 CPU)
    
    12:00:01 AM   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
    12:10:01 AM         2       142      0.00      0.01      0.05         0
    12:20:01 AM         2       142      0.00      0.01      0.05         0
    12:30:01 AM         2       142      0.00      0.01      0.05         0
    12:40:01 AM         3       142      0.00      0.01      0.05         0
    12:50:01 AM         2       142      0.00      0.01      0.05         0
    01:00:01 AM         2       142      0.00      0.01      0.05         0
```

# 问: 在`Linux`下如何指定`dns`服务器，来解析某个域名？

> 答：使用`dig`命令: `dig @DNSip http://domain.com`

如:
```
    dig @8.8.8.8 www.baidu.com#使用谷歌DNS解析百度
```

# 问: 使用`rsync`同步数据时，假如我们采用的是ssh方式，并且目标机器的sshd端口并不是默认的22端口，那我们如何做？

```
    rsync "--rsh=ssh -p 10022"
    
    或
    rsync -e "ssh -p 10022"
```

# 问: `rsync`同步时，如何删除目标数据多出来的数据，即源上不存在，但目标却存在的文件或者目录？

> 答: 加上`--delete`选项

# 问: 使用`free`查看内存使用情况时，哪个数值表示真正可用的内存量？

> 答: `free`列第二行的值

# 问: 有一天你突然发现公司网站访问速度变的很慢很慢，你该怎么办呢？

> 答: 可以从两个方面入手分析
> 1.分析系统负载，使用`w`命令或者`uptime`命令查看系统负载，
> 2.如果负载很高，则使用`top`命令查看`CPU`，`MEM`等占用情况，要么是`CPU`繁忙，要么是内存不够，
如果这二者都正常，再去使用`sar`命令分析网卡流量，分析是不是遭到了攻击。一旦分析出问题的原因，采取对应的措施解决，如决定要不要杀死一些进程，或者禁止一些访问等。

# 问: `rsync`使用服务模式时，如果我们指定了一个密码文件，那么这个密码文件的权限应该设置成多少才可以？

> 答: `600`或`400`