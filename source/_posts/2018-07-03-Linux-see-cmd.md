---
title: Linux 「查看命令」
date: 2018-07-03 10:15:57
categories: Linux cmd
tags: [Linux]
---

> Linux 查询系统信息以及一些别的参数的命令集合

<!-- more -->

# 系统

- `cat /etc/redhat-release` - 查看安装的操作系统版本

```
    [root@izj6c6djex81rijczh0t8yz linux]# cat /etc/redhat-release
    CentOS Linux release 7.5.1804 (Core) 
```

- `uname -a` - 查看内核/操作系统/CPU信息

```php
    [root@izj6c6djex81rijczh0t8yz ~]# uname -a
    Linux izj6c6djex81rijczh0t8yz 3.10.0-862.3.2.el7.x86_64 #1 SMP Mon May 21 23:36:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

- `head -n 1 /etc/issue` - 查看操作系统版本

```php
    [root@izj6c6djex81rijczh0t8yz ~]# head -n 1 /etc/issue
    \S
```

- `cat /proc/cpuinfo` - 查看CPU信息

```php
    [root@izj6c6djex81rijczh0t8yz ~]# cat /proc/cpuinfo
    processor	: 0
    vendor_id	: GenuineIntel
    cpu family	: 6
    model		: 79
    model name	: Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz
    stepping	: 1
    microcode	: 0x1
    cpu MHz		: 2494.222
    cache size	: 40960 KB
    ...
```

- `hostname` - 查看计算机名

```php
    [root@izj6c6djex81rijczh0t8yz ~]# hostname
    izj6c6djex81rijczh0t8yz
```

- `lspci -tv` - 列出所有`PCI`设备

> 安装`lspci`
 - `CentOs`: `yum install pciutils`

```php
    [root@shuidianbang ~]# lspci -tv
    -[0000:00]-+-00.0  Intel Corporation 440FX - 82441FX PMC [Natoma]
               +-01.0  Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
               +-01.1  Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
               +-01.2  Intel Corporation 82371SB PIIX3 USB [Natoma/Triton II]
               +-01.3  Intel Corporation 82371AB/EB/MB PIIX4 ACPI
               +-02.0  Cirrus Logic GD 5446
               +-03.0  Red Hat, Inc Virtio network device
               +-04.0  Red Hat, Inc Virtio console
               +-05.0  Red Hat, Inc Virtio block device
               +-06.0  Red Hat, Inc Virtio block device
               +-07.0  Red Hat, Inc Virtio memory balloon
               \-08.0  Red Hat, Inc Device 1014
```

- `lsusb -tv` - 列出所有`USB`设备

> 安装`lsusb`
  - `CentOs`: `yum install usbutils`
  - `Debian`: `apt-get install usbutils` 

```php
    [root@shuidianbang ~]# lsusb -tv
    /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=uhci_hcd/2p, 12M
        |__ Port 1: Dev 2, If 0, Class=HID, Driver=usbhid, 12M
```

- `lsmod` - 列出加载的内核模块

```
    [root@izj6c6djex81rijczh0t8yz ~]# lsmod
    Module                  Size  Used by
    sb_edac                32034  0 
    iosf_mbi               14990  0 
    crc32_pclmul           13133  0 
    ghash_clmulni_intel    13273  0 
    aesni_intel           189415  0 
    lrw                    13286  1 aesni_intel
    gf128mul               15139  1 lrw
    glue_helper            13990  1 aesni_intel
    ablk_helper            13597  1 aesni_intel
    ...
```

- `env` - 查看环境变量

```php
    [root@izj6c6djex81rijczh0t8yz ~]# env
    XDG_SESSION_ID=1322
    HOSTNAME=izj6c6djex81rijczh0t8yz
    TERM=xterm
    SHELL=/bin/bash
    HISTSIZE=1000
    SSH_CLIENT=61.140.75.213 27033 22
    SSH_TTY=/dev/pts/0
    USER=root
    ...
```

# 资源

- `free -m` - 查看内存使用量和交换区使用量

```php
    [root@izj6c6djex81rijczh0t8yz ~]# free -m
                  total        used        free      shared  buff/cache   available
    Mem:            991         143         156           2         691         674
    Swap:             0           0           0
```

- `df -h` - 查看各分区使用情况

```php
    [root@izj6c6djex81rijczh0t8yz ~]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        40G  4.4G   33G  12% /
    devtmpfs        486M     0  486M   0% /dev
    tmpfs           496M     0  496M   0% /dev/shm
    tmpfs           496M  472K  496M   1% /run
    tmpfs           496M     0  496M   0% /sys/fs/cgroup
    tmpfs           100M     0  100M   0% /run/user/0
```

- `du -sh <目录名>` - 查看指定目录的大小

```php
    [root@izj6c6djex81rijczh0t8yz ~]# 
    [root@izj6c6djex81rijczh0t8yz ~]# du -sh test
    16K	test
```

- `grep MemTotal /proc/meminfo` - 查看内存总量

```php
    [root@izj6c6djex81rijczh0t8yz ~]# grep MemTotal /proc/meminfo
    MemTotal:        1015436 kB
```

- `grep MemFree /proc/meminfo` - 查看空闲内存量

```php
    [root@izj6c6djex81rijczh0t8yz ~]# grep MemFree /proc/meminfo
    MemFree:          160156 kB
```

- `uptime` - 查看系统运行时间、用户数、负载

```php
    [root@izj6c6djex81rijczh0t8yz ~]# uptime
     10:35:49 up 7 days, 17:33,  1 user,  load average: 0.00, 0.01, 0.05
```

- `cat /proc/loadavg` - 查看系统负载

```php
    [root@izj6c6djex81rijczh0t8yz ~]# cat /proc/loadavg
    0.00 0.01 0.05 2/120 22010
```

# 磁盘和分区

- `mount | column -t` - 查看挂接的分区状态

```php
    [root@izj6c6djex81rijczh0t8yz ~]# mount | column -t
    sysfs       on  /sys                             type  sysfs       (rw,nosuid,nodev,noexec,relatime)
    proc        on  /proc                            type  proc        (rw,nosuid,nodev,noexec,relatime)
    devtmpfs    on  /dev                             type  devtmpfs    (rw,nosuid,size=497116k,nr_inodes=124279,mode=755)
    securityfs  on  /sys/kernel/security             type  securityfs  (rw,nosuid,nodev,noexec,relatime)
    tmpfs       on  /dev/shm                         type  tmpfs       (rw,nosuid,nodev)
    devpts      on  /dev/pts                         type  devpts      (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
    tmpfs       on  /run                             type  tmpfs       (rw,nosuid,nodev,mode=755)
    ...
```

- `fdisk -l` - 查看所有分区

```php
    [root@izj6c6djex81rijczh0t8yz ~]# fdisk -l
    
    Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x0008d73a
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/vda1   *        2048    83884031    41940992   83  Linux
```

- `swapon -s` - 查看所有交换分区

```php
    [root@shuidianbang ~]# swapon -s
    Filename				Type		Size	Used	Priority
```

- `hdparm -i /dev/had` - 查看磁盘参数(仅适用于IDE设备)

> 安装`hdparm`
  - `CentOs`: `yum install hdparm -y `

```php
    [root@izj6c6djex81rijczh0t8yz dev]# hdparm -i /dev/disk
    
    /dev/disk:
     HDIO_DRIVE_CMD(identify) failed: Inappropriate ioctl for device
     HDIO_GET_IDENTITY failed: Inappropriate ioctl for device
```

- `dmesg | grep IDE` - 查看启动时IDE设备检测状况

```php
    [root@izj6c6djex81rijczh0t8yz ~]# dmesg | grep IDE
    [    0.123640] pci 0000:00:01.1: legacy IDE quirk: reg 0x10: [io  0x01f0-0x01f7]
    [    0.123642] pci 0000:00:01.1: legacy IDE quirk: reg 0x14: [io  0x03f6]
    [    0.123643] pci 0000:00:01.1: legacy IDE quirk: reg 0x18: [io  0x0170-0x0177]
    [    0.123645] pci 0000:00:01.1: legacy IDE quirk: reg 0x1c: [io  0x0376]
```

# 网络

- `ifconfig` - 查看所有网络接口的属性

```php
    [root@izj6c6djex81rijczh0t8yz ~]# ifconfig
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 172.31.45.94  netmask 255.255.240.0  broadcast 172.31.47.255
            ether 00:16:3e:01:a4:4f  txqueuelen 1000  (Ethernet)
            RX packets 374364  bytes 214011595 (204.0 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 224541  bytes 36447017 (34.7 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 1408  bytes 1195160 (1.1 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 1408  bytes 1195160 (1.1 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

- `iptables -L` - 查看防火墙设置

```php
    [root@izj6c6djex81rijczh0t8yz ~]# iptables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination         
    
    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination         
    
    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination  
```


- `route -n` - 查看路由表

```php
    [root@izj6c6djex81rijczh0t8yz ~]# route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         172.31.47.253   0.0.0.0         UG    0      0        0 eth0
    169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
    172.31.32.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
```

- `netstat -lntp` - 查看所有监听端口

```php
    [root@izj6c6djex81rijczh0t8yz ~]# netstat -lntp
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      2236/nginx: master  
    tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      2256/php-fpm: maste 
    tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1349/mysqld         
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2236/nginx: master  
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1443/sshd 
```

- `netstat -antp` - 查看所有已经建立的连接

```php
    [root@izj6c6djex81rijczh0t8yz ~]# 
    [root@izj6c6djex81rijczh0t8yz ~]# netstat -antp
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      2236/nginx: master  
    tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      2256/php-fpm: maste 
    tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      1349/mysqld         
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      2236/nginx: master  
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1443/sshd           
    tcp        0      0 172.31.45.94:36556      140.205.140.205:80      ESTABLISHED 2798/AliYunDun      
    tcp        0     52 172.31.45.94:22         61.140.75.213:27033     ESTABLISHED 21941/sshd: root@pt 
```

- `netstat -s` - 查看网络统计信息

```php
    [root@izj6c6djex81rijczh0t8yz ~]# netstat -s
    Ip:
        262879 total packets received
        0 forwarded
        0 incoming packets discarded
        262877 incoming packets delivered
        215178 requests sent out
        324 dropped because of missing route
    Icmp:
    ...
```

# 进程

- `ps -ef` - 查看所有进程

```php
    [root@izj6c6djex81rijczh0t8yz ~]# ps -ef
    UID        PID  PPID  C STIME TTY          TIME CMD
    root         1     0  0 Jun25 ?        00:00:05 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
    root         2     0  0 Jun25 ?        00:00:00 [kthreadd]
    root         3     2  0 Jun25 ?        00:00:02 [ksoftirqd/0]
    root         5     2  0 Jun25 ?        00:00:00 [kworker/0:0H]
    root         7     2  0 Jun25 ?        00:00:00 [migration/0]
    ...
```

- `top` - 实时显示进程状态

```php
    top - 10:51:41 up 7 days, 17:48,  1 user,  load average: 0.00, 0.01, 0.05
    Tasks:  69 total,   1 running,  68 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem :  1015436 total,   155664 free,   147448 used,   712324 buff/cache
    KiB Swap:        0 total,        0 free,        0 used.   690456 avail Mem
    ... 
```

# 用户

- `w` - 查看活跃用户

```php
    [root@izj6c6djex81rijczh0t8yz ~]# w
     10:52:27 up 7 days, 17:49,  1 user,  load average: 0.00, 0.01, 0.05
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    root     pts/0    61.140.75.213    10:13    3.00s  0.05s  0.00s w
```

- `id <用户名>` - 查看指定用户信息

```php
    [root@izj6c6djex81rijczh0t8yz ~]# id root
    uid=0(root) gid=0(root) groups=0(root)
    [root@izj6c6djex81rijczh0t8yz ~]# id git
    uid=1002(git) gid=1002(git) groups=1002(git)
```

- `last` - 查看用户登录信息

```php
    [root@izj6c6djex81rijczh0t8yz ~]# last
    root     pts/0        61.140.75.213    Tue Jul  3 10:54   still logged in   
    root     pts/0        61.140.75.213    Tue Jul  3 10:13 - 10:54  (00:41)    
    root     pts/0        61.140.75.213    Tue Jul  3 10:12 - 10:13  (00:01)    
    ...
```

- `cut -d: -f1 /etc/passwd` - 查看系统所有用户

```php
    [root@izj6c6djex81rijczh0t8yz ~]# cut -d: -f1 /etc/passwd
    root
    bin
    daemon
    adm
    lp
    sync
    ...
```

- `cut -d: -f1 /etc/group` - 查看系统所有组

```php
    [root@izj6c6djex81rijczh0t8yz ~]# cut -d: -f1 /etc/group
    root
    bin
    daemon
    sys
    adm
    tty
    disk
    lp
    mem
    ...
```

- `crontab -l` - 查看当前用户的计划任务

```php
    [root@izj6c6djex81rijczh0t8yz ~]# crontab -l
    47 20 * * * curl https://caoxl.com/Aliyun/api_demo/SmsDemo.php
```

# 服务

- `chkconfig --list` - 列出所有系统服务

```php
    [root@shuidianbang ~]# chkconfig --list
    aegis          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    agentwatch     	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    atd            	0:off	1:off	2:off	3:on	4:on	5:on	6:off
    auditd         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    blk-availability	0:off	1:on	2:on	3:on	4:on	5:on	6:off
    cloud-config   	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    cloud-final    	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    ...
```

- `chkconfig --list | grep on` - 列出所有启动的系统服务

```php
    [root@shuidianbang ~]# chkconfig --list | grep on
    aegis          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    agentwatch     	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    atd            	0:off	1:off	2:off	3:on	4:on	5:on	6:off
    auditd         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    blk-availability	0:off	1:on	2:on	3:on	4:on	5:on	6:off
    cloud-config   	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    cloud-final    	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    cloud-init     	0:off	1:off	2:on	3:on	4:on	5:on	6:off
    ...
```


# 程序

- `rpm -qa` - 查看所有安装的软件包

```php
    [root@shuidianbang ~]# rpm -qa
    expat-2.0.1-13.el6_8.x86_64
    python-libs-2.6.6-66.el6_8.x86_64
    basesystem-10.0-4.el6.noarch
    yum-plugin-fastestmirror-1.1.30-40.el6.noarch
    ...
```

- `rpm -qa | grep git` - 查看指定的已安装的软件包

```php
    [root@shuidianbang ~]# rpm -qa | grep git
    xz-4.999.9-0.5.beta.20091007git.el6.x86_64
    git-1.7.1-9.el6_9.x86_64
    xz-libs-4.999.9-0.5.beta.20091007git.el6.x86_64
    libpcap-1.4.0-4.20130826git2dbcaa1.el6.x86_64
    xz-lzma-compat-4.999.9-0.5.beta.20091007git.el6.x86_64
    tcpdump-4.0.0-11.20090921gitdf3cb4.2.el6.x86_64
```
 
# 参考

- [看系统信息的一些命令及查看已安装软件包的命令（转)](http://cheneyph.iteye.com/blog/824746)
- [lspci命令详解](http://coolnull.com/2246.html)
- [在 Linux 中安装使用 lsusb 查看 USB 设备](http://www.wilf.cn/post/lsusb.html)
