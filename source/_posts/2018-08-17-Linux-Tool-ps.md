---
title: Linux ps 进程查看器
date: 2018-08-17 17:46:55
categories: Linux tool
tags: [Linux, ps]
---

> 已经有Linux cmd分类, 为何要新开一个Linux tool分类呢?
> > 因为学习理论是第一位, 但是实际操作更重要, 这个分类将讲Linux下一些工具命令

<!-- more -->

Linux中的`ps`命令是`Process Status`的缩写。`ps`命令用来列出系统中当前运行的那些进程。`ps`命令列出的是当前那些进程的快照，就是执行`ps`命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用`top`命令。

要对进程进行监测和控制，首先必须要了解当前进程的情况，也就是需要查看当前进程，而 ps 命令就是最基本同时也是非常强大的进程查看命令。使用该命令可以确定有**哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源**等等。总之大部分信息都是可以通过执行该命令得到的。

ps 为我们提供了进程的一次性的查看，它所提供的查看结果并不动态连续的；如果想对进程时间监控，应该用 [top linux下的任务管理器] 工具。

# Linux上进程有5种状态:

- **运行**: (正在运行或在运行队列中等待)
- **中断**: (休眠中, 受阻, 在等待某个条件的形成或接受到信号)
- **不可中断**: (收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
- **僵死**: (进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
- **停止*: (进程收到SIGSTOP, SIGTSTP, SIGTTIN, SIGTTOU信号后停止运行运行)

# ps工具标识进程的5种状态码:

- `R 运行` runnable (on run queue)
- `S 中断` sleeping
- `D 不可中断` uninterruptible sleep (usually IO)
- `Z 僵死` a defunct (”zombie”) process
- `T 停止` traced or stopped

# 命令参数

- `a` 显示所有进程
- `-a` 显示同一终端下的所有程序
- `-A` 显示所有进程
- `c` 显示进程的真实名称
- `-N` 反向选择
- `-e` 等于“-A”
- `e` 显示环境变量
- `f` 显示程序间的关系
- `-H` 显示树状结构
- `r` 显示当前终端的进程
- `T` 显示当前终端的所有程序
- `u` 指定用户的所有进程
- `-au` 显示较详细的资讯
- `-aux` 显示所有包含其他使用者的行程
- `-C<命令>` 列出指定命令的状况
- `-C<命令>` 列出指定命令的状况
- `–width<字符数>` 每页显示的字符数
- `–help` 显示帮助信息
- `–version` 显示版本显示

# 输出列的含义

- `F` - 代表这个程序的旗标 (flag)， 4 代表使用者为 super user
- `S` - 代表这个程序的状态 (STAT)，关于各 STAT 的意义将在内文介绍
- `UID` - 程序被该 UID 所拥有
- `PID` - 进程的ID
- `PPID` - 则是其上级父程序的ID
- `C` - CPU 使用的资源百分比
- `PRI` - 这个是 Priority (优先执行序) 的缩写，详细后面介绍
- `NI` - 这个是 Nice 值，在下一小节我们会持续介绍
- `ADDR` - 这个是 kernel function，指出该程序在内存的那个部分。如果是个 running的程序，一般就是 “-“
- `SZ` - 使用掉的内存大小
- `WCHAN` - 目前这个程序是否正在运作当中，若为 - 表示正在运作
- `TTY` - 登入者的终端机位置
- `TIME` - 使用掉的 CPU 时间。
- `CMD` - 所下达的指令为何

# 使用实例

## 实例1：显示所有进程信息

```
    [root@caoxl ~]# ps -A
      PID TTY          TIME CMD
        1 ?        00:00:11 systemd
        2 ?        00:00:00 kthreadd
        3 ?        00:00:04 ksoftirqd/0
        5 ?        00:00:00 kworker/0:0H
        6 ?        00:00:00 kworker/u2:0
        7 ?        00:00:00 migration/0
        8 ?        00:00:00 rcu_bh
        9 ?        00:00:45 rcu_sched
       10 ?        00:00:00 lru-add-drain
```

## 实例2：显示指定用户信息

```
    [root@caoxl ~]# ps -u root
      PID TTY          TIME CMD
        1 ?        00:00:11 systemd
        2 ?        00:00:00 kthreadd
        3 ?        00:00:04 ksoftirqd/0
        5 ?        00:00:00 kworker/0:0H
        6 ?        00:00:00 kworker/u2:0
        7 ?        00:00:00 migration/0
        8 ?        00:00:00 rcu_bh
        9 ?        00:00:45 rcu_sched
       10 ?        00:00:00 lru-add-drain
```

## 实例3：显示所有进程信息，连同命令行

```
    [root@caoxl ~]# ps -ef
    UID        PID  PPID  C STIME TTY          TIME CMD
    root         1     0  0 Aug03 ?        00:00:11 /usr/lib/systemd/systemd --switched-root --sys
    root         2     0  0 Aug03 ?        00:00:00 [kthreadd]
    root         3     2  0 Aug03 ?        00:00:04 [ksoftirqd/0]
    root         5     2  0 Aug03 ?        00:00:00 [kworker/0:0H]
    root         6     2  0 Aug03 ?        00:00:00 [kworker/u2:0]
    root         7     2  0 Aug03 ?        00:00:00 [migration/0]
    root         8     2  0 Aug03 ?        00:00:00 [rcu_bh]
    root         9     2  0 Aug03 ?        00:00:45 [rcu_sched]
    root        10     2  0 Aug03 ?        00:00:00 [lru-add-drain]
```

## 实例4： ps 与grep 组合使用，查找特定进程

```
    [root@caoxl ~]# ps -ef | grep ssh
    root      1476     1  0 Aug03 ?        00:00:00 /usr/sbin/sshd -D
    root     25806  1476  0 10:10 ?        00:00:00 sshd: root@pts/0
    root     26877  1476  0 17:09 ?        00:00:00 sshd: root@pts/1
    root     27031 25808  0 18:03 pts/0    00:00:00 grep --color=auto ssh
```

## 实例5：将与这次登入的 PID 与相关信息列示出来

```
    [root@caoxl ~]# ps -l
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    4 S     0 25808 25806  0  80   0 - 28892 do_wai pts/0    00:00:00 bash
    0 S     0 25899     1  0  80   0 - 581002 futex_ pts/0   00:00:28 java
    0 R     0 27032 25808  0  80   0 - 38300 -      pts/0    00:00:00 ps
```

## 实例6：列出目前所有的正在内存中的程序

```
    [root@caoxl ~]# ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  0.0  0.3  43392  3388 ?        Ss   Aug03   0:11 /usr/lib/systemd/systemd --sw
    root         2  0.0  0.0      0     0 ?        S    Aug03   0:00 [kthreadd]
    root         3  0.0  0.0      0     0 ?        S    Aug03   0:04 [ksoftirqd/0]
    root         5  0.0  0.0      0     0 ?        S<   Aug03   0:00 [kworker/0:0H]
    root         6  0.0  0.0      0     0 ?        S    Aug03   0:00 [kworker/u2:0]
    root         7  0.0  0.0      0     0 ?        S    Aug03   0:00 [migration/0]
    root         8  0.0  0.0      0     0 ?        S    Aug03   0:00 [rcu_bh]
    root         9  0.0  0.0      0     0 ?        R    Aug03   0:45 [rcu_sched]
    root        10  0.0  0.0      0     0 ?        S<   Aug03   0:00 [lru-add-drain]
```