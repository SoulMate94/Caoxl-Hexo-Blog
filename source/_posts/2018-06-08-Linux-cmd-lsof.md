---
title: Linux 命令 「lsof」
date: 2018-06-08 09:28:45
categories: Linux cmd
tags: [Linux]
---

# `lsof`

**lsof命令**用于查看你进程开打的文件，打开文件的进程，进程打开的端口(TCP、UDP)。
找回/恢复删除的文件。是十分方便的系统监视工具，因为`lsof`命令需要访问核心内存和各种文件，所以需要root用户执行。

<!-- more -->

# 命令参数

- `-a`: 列出打开文件存在的进程
- `-c`<进程>: 列出指定进程所打开的文件
- `-g`: 列出GID进程详情
- `-d`<文件号>: 列出占用该文件号的进程
- `+d`<目录>: 列出目录下被打开的文件
- `+D`<目录>: 递归列出目录下被打开的文件
- `-n`<目录>: 列出使用NFS的文件
- `-i`<条件>: 列出符合条件的进程. (4,6,协议,:端口,@ip)
- `-p`<进程>: 列出指定进程所打开的文件
- `-u`: 列出UID号进程
- `-h`: 显示帮助信息
- `-v`: 显示版本信息

# 实例

```linux
    [root@izj6c6djex81rijczh0t8yz ~]# lsof -i
    COMMAND     PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
    ntpd        886   ntp   16u  IPv4  13635      0t0  UDP *:ntp 
    ntpd        886   ntp   17u  IPv6  13637      0t0  UDP *:ntp 
    ntpd        886   ntp   18u  IPv4  13642      0t0  UDP localhost:ntp 
    ntpd        886   ntp   19u  IPv4  13643      0t0  UDP izj6c6djex81rijczh0t8yz:ntp 
    mysqld     1464 mysql   11u  IPv4  14633      0t0  TCP *:mysql (LISTEN)
    nginx      1518  root    8u  IPv4  14491      0t0  TCP *:http (LISTEN)
    nginx      1518  root    9u  IPv4  14492      0t0  TCP *:https (LISTEN)
    nginx      1519   www    8u  IPv4  14491      0t0  TCP *:http (LISTEN)
    nginx      1519   www    9u  IPv4  14492      0t0  TCP *:https (LISTEN)
    sshd       1567  root    3u  IPv4  15069      0t0  TCP *:ssh (LISTEN)
    redis-ser  5466  root    6u  IPv4 869898      0t0  TCP localhost:6379 (LISTEN)
    sshd       9860  root    3u  IPv4 908787      0t0  TCP izj6c6djex81rijczh0t8yz:ssh->61.140.75.173:19681 (ESTABLISHED)
    AliYunDun 28741  root   18u  IPv4 904389      0t0  TCP izj6c6djex81rijczh0t8yz:53014->106.11.68.13:http (ESTABLISHED)
```

## 参数解释

- `COMMAND`: 进程的名称
- `PID`: 进程标识符
- `PPID`: 父进程标识符 (需要指定 -R 参数)
- `USER`: 进程所有者
- `PGID`: 进程所属组
- `FD`: 文件描述符, 应用程序通过文件描述识别该文件

### 文件描述符列表

- `cwd`: current work directory,即:应用程序的当前工作目录
- `txt`: 该类型的文件是程序代码
- `lnn`: 参考文献
- `er`: FD信息误差(参见名称栏)
- `jld`: 监狱目录( FreeBSD一种可免费使用的UNIX操作系统)
- `ltx`: 共享库文本(代码和数据)
- `mxx`: 十六进制内存映射类型XX。
- `m86`: DOS合并映射文件
- `mem`: 内存映射文件
- `mmap`: 内存映射设备
- `pd`: 父目录
- `rtd`: 根目录
- `tr`: 内核跟踪文件(OpenBSD)
- `v86`:  VP/ix 隐射文件
- `0`: 表示标准输出
- `1`: 表示标准输入
- `2`: 表示标准错误

**一般在标准输出、标准错误、标准输入后还跟着文件状态模式：**

- `u`: 表示该文件被打开并处于读取/写入模式
- `r`: 表示该文件被打开并处于只读模式
- `w`: 表示该文件被打开并处于写入模式
- `空格`: 表示该文件的状态模式为 `unknown`,且没有锁定
- `-`: 表示该文件的状态模式为 `unknown`, 且被锁定

**同时在文件状态模式后面，还跟着相关的锁：**

- `N`: 对于未知类型的Solaris NFS锁
- `r`: 文件的部分读锁
- `R`: 整个文件的读锁
- `w`: 文件的部分写锁
- `W`: 整个文件的写锁
- `u`: 对于任意长度的读写锁
- `U`: 对于未知类型的锁
- `x`: 对于SCO OpenServer XIX锁的一部分文件
- `X`: 对于SCO OpenServer XIX锁的整个文件
- `空格`: 没有锁

### 文件类型

- `DIR`: 目录
- `CHR`: 字符类型
- `BLK`: 块设备类型
- `UNIX`: UNIX域套接字
- `FIFO`: 先进先出队列
- `IPv4`: 网际协议(IP)套接字
- `DEVICE`: 指定磁盘的名称
- `SIZE`: 文件的大小
- `NODE`: 索引节点 (文件在磁盘上的标识)
- `NAME`: 打开文件的确切名称