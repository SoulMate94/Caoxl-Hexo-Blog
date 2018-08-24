---
title: Linux 系统管理及IPC资源管理
date: 2018-08-24 10:21:22
categories: Linux
tags: [Linux, 系统管理, IPC]
---

> Linux 系统管理及IPC资源管理

<!-- more -->


# 系统管理

## 查询系统版本

- 查看Linux系统版本:

```
    [root@caoxl ~]# uname -a
    Linux caoxl 3.10.0-862.9.1.el7.x86_64 #1 SMP Mon Jul 16 16:29:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
    
    # 或者
    lsb_release -a
```

## 查询硬件信息

- 查看CPU使用情况:

```
    [root@caoxl ~]# sar -u 5 10
    Linux 3.10.0-862.9.1.el7.x86_64 (caoxl) 	08/24/2018 	_x86_64_	(1 CPU)
    
    10:26:12 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
    10:26:17 AM     all      0.20      0.00      0.20      0.00      0.00     99.60
    10:26:22 AM     all      0.00      0.00      0.20      0.00      0.00     99.80
    10:26:27 AM     all      0.40      0.00      0.20      0.00      0.00     99.40
    10:26:32 AM     all      0.20      0.00      0.00      0.80      0.00     99.00
    10:26:37 AM     all      0.20      0.00      0.20      0.00      0.00     99.60
    10:26:42 AM     all      0.20      0.00      0.20      0.00      0.00     99.60
    10:26:47 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
    10:26:52 AM     all      0.20      0.00      0.00      0.00      0.00     99.80
    10:26:57 AM     all      0.20      0.00      0.20      0.00      0.00     99.60
    10:27:02 AM     all      0.20      0.00      0.20      0.00      0.00     99.60
    Average:        all      0.18      0.00      0.14      0.08      0.00     99.60
```

- 查询CPU信息:

```
    [root@caoxl ~]# cat /proc/cpuinfo
    processor	: 0
    vendor_id	: GenuineIntel
    cpu family	: 6
    model		: 79
    model name	: Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz
    stepping	: 1
    microcode	: 0x1
    cpu MHz		: 2499.996
    cache size	: 40960 KB
    physical id	: 0
    siblings	: 1
    core id		: 0
    cpu cores	: 1
    apicid		: 0
    initial apicid	: 0
    fpu		: yes
    fpu_exception	: yes
    cpuid level	: 13
    wp		: yes
    flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss syscall nx pdpe1gb rdtscp lm constant_tsc rep_good nopl eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch ibrs ibpb stibp fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm rdseed adx smap xsaveopt spec_ctrl intel_stibp
    bogomips	: 4999.99
    clflush size	: 64
    cache_alignment	: 64
    address sizes	: 46 bits physical, 48 bits virtual
    power management:
```

- 查看CPU的核的个数:

```
    [root@caoxl ~]# cat /proc/cpuinfo | grep processor | wc -l
    1
```

- 查看内存信息:

```
    [root@caoxl ~]# cat /proc/meminfo
    MemTotal:        1015436 kB
    MemFree:          117588 kB
    MemAvailable:     526340 kB
    Buffers:           77540 kB
    Cached:           426928 kB
    SwapCached:            0 kB
    Active:           641068 kB
    Inactive:         138596 kB
    Active(anon):     275492 kB
    Inactive(anon):     2368 kB
    Active(file):     365576 kB
    Inactive(file):   136228 kB
    Unevictable:           0 kB
    Mlocked:               0 kB
    SwapTotal:             0 kB
    SwapFree:              0 kB
    Dirty:                16 kB
    Writeback:             0 kB
    AnonPages:        275228 kB
    Mapped:            37868 kB
    Shmem:              2664 kB
    Slab:              84628 kB
    SReclaimable:      69448 kB
    SUnreclaim:        15180 kB
    KernelStack:        2928 kB
    PageTables:        10428 kB
    NFS_Unstable:          0 kB
    Bounce:                0 kB
    WritebackTmp:          0 kB
    CommitLimit:      507716 kB
    Committed_AS:     890020 kB
    VmallocTotal:   34359738367 kB
    VmallocUsed:        8812 kB
    VmallocChunk:   34359719676 kB
    HardwareCorrupted:     0 kB
    AnonHugePages:     57344 kB
    CmaTotal:              0 kB
    CmaFree:               0 kB
    HugePages_Total:       0
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    DirectMap4k:       63360 kB
    DirectMap2M:      985088 kB
    DirectMap1G:           0 kB
```

- 显示内存page大小（以KByte为单位）:

```
    [root@caoxl ~]# pagesize
    4096
    
    # 安装pagesize
    [root@caoxl ~]# yum search pagesize
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
    ===================================== Matched: pagesize ======================================
    libhugetlbfs-utils.x86_64 : Userspace utilities for configuring the hugepage environment

    [root@caoxl ~]# yum install -y libhugetlbfs-utils
```

- 显示架构:

```
    [root@caoxl ~]# arch
    x86_64
```

## 设置系统时间

- 显示当前系统时间:

```
    [root@caoxl ~]# date
    Fri Aug 24 10:31:05 CST 2018
```

- 设置系统日期和时间(格式为2018-8-24 10:30:00):

```
    date -s 2018-8-24 10:30:00
    date -s 2018-8-24
    date -s 10:30:00
```

- 设置时区:

```
    选择时区信息。命令为：tzselect
    根据系统提示，选择相应的时区信息。
    
    [root@caoxl ~]# tzselect
    Please identify a location so that time zone rules can be set correctly.
    Please select a continent or ocean.
     1) Africa
     2) Americas
     3) Antarctica
     4) Arctic Ocean
     5) Asia
     6) Atlantic Ocean
     7) Australia
     8) Europe
     9) Indian Ocean
    10) Pacific Ocean
    11) none - I want to specify the time zone using the Posix TZ format.
```

- 强制把系统时间写入CMOS（这样，重启后时间也正确了）:

```
    [root@caoxl ~]# clock -w
```

> 设置系统时间需要root用户权限.

- 格式化输出当前日期时间:

```
   [root@caoxl ~]# date +%Y%m%d.%H%M%S
   20180824.103400 
```

# IPC资源管理

## IPC资源查询

- 查看系统使用的IPC资源:

```
    [root@caoxl ~]# ipcs
    
    ------ Message Queues --------
    key        msqid      owner      perms      used-bytes   messages    
    
    ------ Shared Memory Segments --------
    key        shmid      owner      perms      bytes      nattch     status      
    
    ------ Semaphore Arrays --------
    key        semid      owner      perms      nsems  
```

- 查看系统使用的IPC共享内存资源:

```
    [root@caoxl ~]# ipcs -m
    
    ------ Shared Memory Segments --------
    key        shmid      owner      perms      bytes      nattch     status 
```

- 查看系统使用的IPC队列资源:

```
    [root@caoxl ~]# ipcs -q
    
    ------ Message Queues --------
    key        msqid      owner      perms      used-bytes   messages    
```

- 查看系统使用的IPC信号量资源:

```
    [root@caoxl ~]# ipcs -s
    
    ------ Semaphore Arrays --------
    key        semid      owner      perms      nsems 
```

应用示例：查看IPC资源被谁占用

有个IPCKEY：51036 ，需要查询其是否被占用；

1. 首先通过计算器将其转为十六进制:

> 51036 -> c75c

2. 如果知道是被共享内存占用:

```
    [root@caoxl ~]# ipcs -m | grep c75c
```

3. 如果不确定，则直接查找:

```
    [root@caoxl ~]# ipcs | grep c75c
```

## 检测和设置系统资源限制

- 显示当前所有的系统资源 `limit` 信息:

```
    [root@caoxl ~]# ulimit -a
    core file size          (blocks, -c) 0
    data seg size           (kbytes, -d) unlimited
    scheduling priority             (-e) 0
    file size               (blocks, -f) unlimited
    pending signals                 (-i) 3883
    max locked memory       (kbytes, -l) 64
    max memory size         (kbytes, -m) unlimited
    open files                      (-n) 65535
    pipe size            (512 bytes, -p) 8
    POSIX message queues     (bytes, -q) 819200
    real-time priority              (-r) 0
    stack size              (kbytes, -s) 8192
    cpu time               (seconds, -t) unlimited
    max user processes              (-u) 65535
    virtual memory          (kbytes, -v) unlimited
    file locks                      (-x) unlimited
```

- 对生成的 core 文件的大小不进行限制:

```
    [root@caoxl ~]# ulimit -c unlimited
```

# 总结

> uname sar arch date ipcs ulimit