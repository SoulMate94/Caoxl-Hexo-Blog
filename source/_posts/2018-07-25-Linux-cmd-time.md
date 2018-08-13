---
title: Linux 命令 「time」
date: 2018-07-25 17:30:16
categories: Linux cmd
tags: [Linux]
---

> `time`命令用于测算一个命令（即程序）的执行时间

<!-- more -->

# 语法

```
    time command
```

# 实例

```
    [root@caoxianliang /]# time ls
    bin   data  etc   lib    lnmp1.4     media  opt   root  sbin  sys  usr
    boot  dev   home  lib64  lost+found  mnt    proc  run   srv   tmp  var
    
    real	0m0.002s
    user	0m0.000s
    sys	    0m0.002s
```

**说明:**

- `real`: 实际实际, 从command命令行开始执行到运行终止的消逝时间
- `user`: 用户CPU时间, 命令执行完成花费的用户CPU时间，即命令在用户态中执行时间总和
- `sys`:  系统CPU时间，命令执行完成花费的系统CPU时间，即命令在核心态中执行时间总和

> 用户CPU时间和系统CPU时间之和为CPU时间，即命令占用CPU执行的时间总和。**实际时间要大于CPU时间**，因为Linux是多任务操作系统，往往在执行一条命令时，系统还要处理其它任务。另一个需要注意的问题是即使每次执行相同命令，但所花费的时间也是不一样，其花费时间是与系统运行相关的。

