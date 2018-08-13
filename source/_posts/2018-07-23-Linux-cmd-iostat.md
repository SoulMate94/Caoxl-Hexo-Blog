---
title: Linux 命令 「iostat」
date: 2018-07-23 14:39:13
categories: Linux cmd
tags: [Linux]
---

> **iostat命令**被用于监视系统输入输出设备和CPU的使用情况。

<!-- more -->

它的特点是汇报磁盘活动统计情况，同时也会汇报出CPU使用情况。同`vmstat`一样，`iostat`也有一个弱点，就是它不能对某个进程进行深入分析，仅对系统的整体情况进行分析

# 语法

```
    iostat (选项) (参数)
```

# 选项

- `-c`: 仅显示CPU使用情况
- `-d`: 仅显示设备利用率
- `-k`: 显示状态以千字节每秒为单位, 而不使用块每秒
- `-m`: 显示状态以兆字节每秒为单位,
- `-p`: 仅显示块设备和所有被使用的其他分区的状态
- `-t`: 显示每个报告产生的时间
- `-V`: 显示版号并退出
- `-x`: 显示扩展状态

# 参数

- 间隔时间: 每次报告的间隔时间 (秒)
- 次数: 显示报告的次数

# 实例

## 用 `iostat -x /dev/sda1` 来观看磁盘 `I/O` 的详细情况:

```
    [root@caoxianliang ~]# iostat -x /dev/sda1
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.20    0.01    0.11    0.02    0.00   99.65
    
    Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
```

详细说明: **第二行是系统信息和监测时间**, **第三行和第四行显示CPU使用情况**（具体内容和`mpstat`命令相同）。这里主要关注后面I/O输出的信息，如下所示:

**参数说明:**

### `cpu`属性值说明

- `%user` - CPU处在用户模式下的时间百分比
- `%nice` - CPU处在带NICE值的用户模式下的时间百分比
- `%system` - CPU处在系统模式下的时间百分比
- `%iowait` - CPU等待输入输出完成时间的百分比
- `%steal` - 管理程序维护另一个虚拟处理器时, 虚拟CPU的无意识等待时间百分比
- `%idle` - CPU空闲时间百分比

### `disk`属性值说明

- `Device` - 检测设备名称
- `rrqm/s` - 每秒需要读取需求的数量
- `wrqm/s` - 每秒需要写入需求的数量
- `r/s` - 每秒实际读取需求的数量
- `w/s` - 每秒实际写入需求的数量
- `rsec/s` - 每秒读取区段的数量
- `wsec/s` - 每秒写入区段的数量
- `rkB/s` - 每秒实际读取的大小, 单位为`KB`
- `wkB/s` - 每秒实际写入的大小, 单位为`KB`
- `avgqu-sz` - 需求的平均大小区段
- `avgqu-sz` - 需求的评价队列长度
- `await` - 等待`I/O`平均的时间
- `svctm` - `I/O`需求完成的平均时间
- `%util` - 被`I/O`需求消耗的`CPU`百分比


## 定时显示所有信息

```
    [root@caoxianliang ~]# iostat 2 3
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.20    0.01    0.11    0.02    0.00   99.65
    
    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    vda               0.85         1.55         5.30    1327517    4544968
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.00    0.00    0.00    0.00    0.00  100.00
    
    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    vda               0.00         0.00         0.00          0          0
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.00    0.00    0.00    0.00    0.00  100.00
    
    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    vda               0.00         0.00         0.00          0          0
```

> 每隔 2秒刷新显示，且显示3次

## 显示指定磁盘信息

- `iostat -d sda1`

```
    [root@caoxianliang ~]# iostat -d sda1
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
```

## 显示`tty`和`cpu`信息

```
    [root@caoxianliang ~]# iostat -t
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    07/23/2018 03:48:46 PM
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.20    0.01    0.11    0.02    0.00   99.65
    
    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    vda               0.85         1.55         5.30    1327517    4544988
```

## 以M为单位显示所有信息

```
    [root@caoxianliang ~]# iostat -m
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.20    0.01    0.11    0.02    0.00   99.66
    
    Device:            tps    MB_read/s    MB_wrtn/s    MB_read    MB_wrtn
    vda               0.85         0.00         0.01       1296       4438
```

## 查看`TPS`和吞吐量信息

```
    [root@caoxianliang ~]# iostat -d -k 1 1
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    vda               0.85         1.55         5.30    1327517    4544988
```

## 查看设备使用率(%util)、响应时间(await)

```
    [root@caoxianliang ~]# iostat -d -x -k 1 1
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
    vda               0.00     0.34    0.09    0.76     1.55     5.30    16.10     0.00    5.52    7.06    5.34   0.33   0.03
```

## 查看CPU状态

```
    [root@caoxianliang ~]# iostat -c 1 3
    Linux 3.10.0-862.3.2.el7.x86_64 (caoxianliang) 	07/23/2018 	_x86_64_	(1 CPU)
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.20    0.01    0.11    0.02    0.00   99.66
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.00    0.00    0.00    0.00    0.00  100.00
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.00    0.00    0.00    0.00    0.00  100.00
```

