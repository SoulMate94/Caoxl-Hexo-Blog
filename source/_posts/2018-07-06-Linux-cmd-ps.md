---
title: Linux 命令 「ps」
date: 2018-07-06 14:08:39
categories: Linux cmd
tags: [Linux]
---

> **ps命令**用于报告当前系统的进程状态

<!-- more -->

> 可以搭配`kill`指令随时中断、删除不必要的程序.
ps命令是最基本同时也是非常强大的进程查看命令，使用该命令可以确定有哪些进程正在运行和运行的状态,
进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等，总之大部分信息都是可以通过执行该命令得到的。

# 语法

```
    ps (选项)
```

# 选项

- `-a`:   显示所有进程
- `a`:    显示同一终端下的所有程序
- `-A`:   显示所有程序
- `-c`:   显示`CLS`和`PRI`栏位
- `c`:    显示进程的真实名称
- `-C<指令名称>`:   指定执行指令的名称, 并列出该指令的程序的状况
- `-d`:   显示所有程序, 但不包括阶段作业领导者的程序
- `-e`:   此选项的效果和指定`A`选项相同
- `e`:    列出程序时, 显示每个程序所使用的环境变量
- `f`:    显示程序间的关系
- `-H`:   显示树状结构
- `r`:    显示当前终端的所有程序
- `T`:    显示当前终端的所有程序
- `u`:    指定用户的所有进程
- `-au`:  显示较详细的资讯
- `-aux`: 显示所有包含其他使用者的进程
- `--lines<行数>`: 每页显示的行数
- `--width<字符数>`: 每页显示的字符数
- `--help`: 显示帮助信息
- `--version`: 显示版本显示   


> 由于`ps`命令能够支持的系统类型相当的多，所以选项多的离谱！ 详情:[ps](http://man.linuxde.net/ps)

## 常见命令

- **显示所有进程的信息 - `ps -A`**

```
     [root@izj6c6djex81rijczh0t8yz ~]# ps -A
     PID TTY          TIME CMD
       1 ?        00:00:07 systemd
       2 ?        00:00:00 kthreadd
       3 ?        00:00:03 ksoftirqd/0
       5 ?        00:00:00 kworker/0:0H
       7 ?        00:00:00 migration/0
     ... 
```

- **显示指定用户信息 - `ps -u root`**

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps -u root
      PID TTY          TIME CMD
        1 ?        00:00:07 systemd
        2 ?        00:00:00 kthreadd
        3 ?        00:00:03 ksoftirqd/0
        5 ?        00:00:00 kworker/0:0H
        7 ?        00:00:00 migration/0
    ...

    [root@izj6c6djex81rijczh0t8yz ~]# ps -u git
      PID TTY          TIME CMD
```

- **显示所有进程信息, 连同命令行 - `ps -ef`**

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps -ef
    UID        PID  PPID  C STIME TTY          TIME CMD
    root         1     0  0 Jun25 ?        00:00:07 /usr/lib/systemd/systemd --switched-root --system --deserializ
    root         2     0  0 Jun25 ?        00:00:00 [kthreadd]
    root         3     2  0 Jun25 ?        00:00:03 [ksoftirqd/0]
    root         5     2  0 Jun25 ?        00:00:00 [kworker/0:0H]
    root         7     2  0 Jun25 ?        00:00:00 [migration/0]
    ...
```

- **`ps` 与 `grep` 常用组合用户, 查找特地进程 - `ps -ef | grep ssh`**

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps -ef | grep ssh
    root       449 32147  0 14:47 pts/0    00:00:00 grep --color=auto ssh
    root      1443     1  0 Jun25 ?        00:00:04 /usr/sbin/sshd -D
    root     32145  1443  0 09:49 ?        00:00:00 sshd: root@pts/0
```

- **将目前属于你自己这次登入的`PID`与相关信息列示出来 - `ps -l`**

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps -l
    F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    0 R     0   450 32147  0  80   0 - 38292 -      pts/0    00:00:00 ps
    4 S     0 32147 32145  0  80   0 - 28892 do_wai pts/0    00:00:00 bash
```

**说明:**

- `F`:      代表正呈现的`flag`, 4 代表使用者为 `super user`
- `S`:      代表这个程序的状态(STAT)
- `UID`:    程序拥有者
- `PID`:    程序的ID
- `PPID`:   上级父程序的ID
- `C`:      CPU使用的资源百分比
- `PRI`:    `Priority`(优先执行序)的简写
- `NI`:     `Nice`值
- `ADDR`:   `kernel function`指出该程序在内存的哪个部分。如果是个 `running`的程序，一般就是 "-"
- `SZ`:     已使用的内存大小
- `WCHEAN`: 是否正在运行, `-`表示正在运行
- `TTY`:    登入者的终端机位置
- `TIME`:   已使用的`CPU`时间
- `CMD`:    所下达的指令名称

> 在预设的情况下， `ps` 仅会列出与目前所在的 `bash shell` 有关的 `PID` 而已，所以， 当我使用 `ps -l` 的时候，只有2个 `PID`

- **列出目前所有的正在内存当中的程序 - `ps aux`**

```
   [root@izj6c6djex81rijczh0t8yz ~]# ps aux
   USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
   root         1  0.0  0.3  43404  3140 ?        Ss   Jun25   0:07 /usr/lib/systemd/systemd --switched-root --sy
   root         2  0.0  0.0      0     0 ?        S    Jun25   0:00 [kthreadd]
   root         3  0.0  0.0      0     0 ?        S    Jun25   0:03 [ksoftirqd/0]
   root         5  0.0  0.0      0     0 ?        S<   Jun25   0:00 [kworker/0:0H]
   root         7  0.0  0.0      0     0 ?        S    Jun25   0:00 [migration/0] 
```

**说明:**

- `USER`:   该 `process` 属于哪个使用者账号的
- `PID`:    该 `process` 的`ID`
- `%CPU`:   该 `process` 使用掉的 `CPU` 资源百分比
- `%MEM`:   该 `process` 所占用的物理内存百分比
- `VSZ`:    该 `process` 使用掉的虚拟内存量 (Kbytes)
- `RSS`:    该 `process` 占用的固定的内存量 (Kbytes)
- `TTY`:    该 `process` 是在哪个终端机上面运作，若与终端机无关，则显示 `?`，另外， `tty1-tty6` 是本机上面的登入者程序，若为 `pts/0` 等等的，则表示为由网络连接进主机的程序
- `STAT`:   该程序目前的状态，主要的状态有
  - `R`: 该程序目前正在运作，或者是可被运作
  - `S`: 该程序目前正在睡眠当中 (可说是 `idle` 状态)，但可被某些讯号 (`signal`) 唤醒
  - `T`: 该程序目前正在侦测或者是停止了
  - `Z`: 该程序应该已经终止，但是其父程序却无法正常的终止他，造成 `zombie` (僵尸) 程序的状态
- `START`:  该 `process` 被触发启动的时间
- `TIME`:   该 `process` 实际使用 `CPU` 运作的时间
- `COMMAND`: 该程序的实际指令


- **列出类似程序树的程序显示 - `ps -axjf`**

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps -axjf
     PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
        0     2     0     0 ?           -1 S        0   0:00 [kthreadd]
        2     3     0     0 ?           -1 S        0   0:03  \_ [ksoftirqd/0]
        2     5     0     0 ?           -1 S<       0   0:00  \_ [kworker/0:0H]
        2     7     0     0 ?           -1 S        0   0:00  \_ [migration/0]
        2     8     0     0 ?           -1 S        0   0:00  \_ [rcu_bh]
        2     9     0     0 ?           -1 R        0   0:31  \_ [rcu_sched]
```


- **找出与`cron`与`syslog`这个两个服务有关的`PID`号码 - `ps aux | egrep '(cron|syslog)'`**

```
    [root@izj6c6djex81rijczh0t8yz ~]# ps aux | egrep '(cron|syslog)'
    root       471  0.0  0.1 126316  1644 ?        Ss   Jun25   0:01 /usr/sbin/crond -n
    root       546  0.0  0.0 112708  1000 pts/0    R+   15:24   0:00 grep -E --color=auto (cron|syslog)
    root       706  0.0  1.8 548532 18440 ?        Ssl  Jun25   0:53 /usr/sbin/rsyslogd -n
```

