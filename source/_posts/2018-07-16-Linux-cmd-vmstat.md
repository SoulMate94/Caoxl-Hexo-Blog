---
title: Linux 命令 「vmstat」
date: 2018-07-16 10:19:07
categories: Linux cmd
tags: [Linux]
---

> **vmstat命令**的含义为显示虚拟内存状态 (`“Viryual Memor Statics”`), 但是它可以报告关于进程、内存、I/O等系统整体运行状态

<!-- more -->

# 语法

```
    vmstat (选项) (参数)
```

# 选项

- `-a`: 显示活动内页
- `-f`: 显示启动后创建的进程总数
- `-m`: 显示`slab`信息
- `-n`: 头信息仅显示一次
- `-s`: 以表格方式显示事件计数器和内存状态
- `-d`: 报告磁盘状态
- `-p`: 显示指定的硬盘分区状态
- `-S`: 输出信息的单位

# 参数

- 事件间隔: 状态信息刷新的时间间隔
- 次数: 显示报告的次数

# 实例

```
    [root@caoxianliang ~]# vmstat
    procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     3  0      0 103704  99284 671468    0    0     2     6   87  158  0  0 100  0  0
```

**字段说明:**

- `Procs (进程)`
  - `r`: 运行队列中进程数量, 这个值也可以判断是否需要增加CPU (长期大于1)
  - `b`: 等待IO的进程数量
  
- `Memory (内存)`
  - `swpd`: 使用虚拟内存大小, 如果`swpd`的值不为0, 但是`SI`,`SO`的值长期为0, 这种情况不糊影响系统性能
  - `free`: 空闲物理内存大小
  - `buff`: 用作缓冲的内存大小
  - `cache`: 用作缓冲的内存大小, 如果`cache`的值大的时候, 说明`cache`处的文件数多, 如果频繁访问到的文件都能被`cache`处，那么磁盘的读`IO bi`会非常小。
  
- `Swap`
  - `si`: 每秒从交换区写到内存大小, 由磁盘调入内存
  - `so`: 每秒写入交换区的内存大小, 由内存调入磁盘

> 注意: **内存够用的时候，这2个值都是0*, 如果这2个值长期大于0时，系统性能会受到影响，磁盘IO和CPU资源都会被消耗。有些朋友看到空闲内存（free）很少的或接近于0时，就认为内存不够用了，不能光看这一点，还要结合si和so, **如果free很少，但是si和so也很少（大多时候是0），那么不用担心，系统性能这时不会受到影响的**

- `IO (现在的Linux版本块的大小为1kb)`
  - `bi`: 每秒读取的块数
  - `bo`: 每秒写入的块数

> 注意：随机磁盘读写的时候，这2个值越大（如超出1024k)，能看到CPU在IO等待的值也会越大。

- `System (系统)`
  - `in`: 每秒中断数, 包括时钟中断
  - `cs`: 每秒上下文切换数
  
> 注意: 上面2个值越大, 会看到由内核消耗的CPU时间会越大.

- `CPU` (以百分比表示)
  - `us`: 用户进程执行时间百分比(`user time`)
  > `us`的值比较高时, 说明用户进程消耗的`CPU`时间多，但是如果长期超`50%`的使用，那么我们就该考虑优化程序算法或者进行加速;
  - `sy`: 内核系统进程执行时间百分比 (`system time`)
  > `sy`的值高时, 说明系统内核消耗的CPU资源多，这并不是良性表现，我们应该检查原因;
  - `wa`: `IO`等待时间百分比
  > `wa`的值高时，说明`IO`等待比较严重，这可能由于磁盘大量作随机访问造成，也有可能磁盘出现瓶颈（块操作）。
  - `id`: 空闲时间百分比
