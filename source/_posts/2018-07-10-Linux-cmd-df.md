---
title: Linux 命令 「df」
date: 2018-07-10 17:25:02
categories: Linux cmd
tags: [Linux]
---

> `df`命令用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为`KB`。

<!-- more -->

可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

# 语法

```
    df (选项) (参数)
```

# 选项

- `-a`/`--a`:   包含全部的文件系统
- `--block-size=<区块大小>`: 以指定的区块大小来显示区块数目
- `-h`/`--human-readable`: 以可读性较高的方式来显示信息
- `-H`/`--si`: 与`-h`参数相同，但在计算时是以`1000` Bytes为换算单位而非`1024` Bytes
- `-i`/`--inodes`: 显示`inode`的信息
- `-k`/`--kilobytes`: 指定区块大小为`1024`字节
- `-l`/`--local`: 仅显示本地端的文件系统
- `-m`/`--megabytes`: 指定区块大小为`1048576`字节
- `--no-sync`: 在取得磁盘使用信息前，不要执行`sync`指令，此为预设值
- `-t<文件系统类型>`/`--type=<文件系统类型>`: 仅显示指定文件系统类型的磁盘信息
- `-x<文件系统类型>`/`--exclude-type=<文件系统类型>`: 不要显示指定文件系统类型的磁盘信息
- `-T`/`--print-type`: 显示文件系统的类型
- `--help`: 显示帮助
- `--version`: 显示版本信息

# 参数

- 文件: 指定文件系统上的文件

# 实例

- 查看系统磁盘设备，默认是KB为单位:

```
    [root@izj6c6djex81rijczh0t8yz ~]# df
    # 文件系统        1K-块       已用   可用      已用% 挂载点
    Filesystem     1K-blocks    Used Available Use% Mounted on
    /dev/vda1       41151808 6917616  32120760  18% /
    devtmpfs          497116       0    497116   0% /dev
    tmpfs             507716       0    507716   0% /dev/shm
    tmpfs             507716     472    507244   1% /run
    tmpfs             507716       0    507716   0% /sys/fs/cgroup
    tmpfs             101544       0    101544   0% /run/user/0
```

- 使用`-h`选项以`KB`以上的单位来显示，可读性高

```
    [root@izj6c6djex81rijczh0t8yz ~]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        40G  6.6G   31G  18% /
    devtmpfs        486M     0  486M   0% /dev
    tmpfs           496M     0  496M   0% /dev/shm
    tmpfs           496M  472K  496M   1% /run
    tmpfs           496M     0  496M   0% /sys/fs/cgroup
    tmpfs           100M     0  100M   0% /run/user/0
```

- 查看全部文件系统:

```
    [root@izj6c6djex81rijczh0t8yz ~]# df -a
    Filesystem     1K-blocks    Used Available Use% Mounted on
    rootfs                 -       -         -    - /
    sysfs                  0       0         0    - /sys
    proc                   0       0         0    - /proc
    devtmpfs          497116       0    497116   0% /dev
    securityfs             0       0         0    - /sys/kernel/security
    tmpfs             507716       0    507716   0% /dev/shm
    devpts                 0       0         0    - /dev/pts
    tmpfs             507716     472    507244   1% /run
    tmpfs             507716       0    507716   0% /sys/fs/cgroup
    cgroup                 0       0         0    - /sys/fs/cgroup/systemd
    pstore                 0       0         0    - /sys/fs/pstore
    cgroup                 0       0         0    - /sys/fs/cgroup/cpuset
    cgroup                 0       0         0    - /sys/fs/cgroup/net_cls,net_prio
    cgroup                 0       0         0    - /sys/fs/cgroup/hugetlb
    cgroup                 0       0         0    - /sys/fs/cgroup/memory
    cgroup                 0       0         0    - /sys/fs/cgroup/blkio
    cgroup                 0       0         0    - /sys/fs/cgroup/freezer
    cgroup                 0       0         0    - /sys/fs/cgroup/devices
    cgroup                 0       0         0    - /sys/fs/cgroup/perf_event
    cgroup                 0       0         0    - /sys/fs/cgroup/cpu,cpuacct
    cgroup                 0       0         0    - /sys/fs/cgroup/pids
    configfs               0       0         0    - /sys/kernel/config
    /dev/vda1       41151808 6917748  32120628  18% /
    systemd-1              -       -         -    - /proc/sys/fs/binfmt_misc
    hugetlbfs              0       0         0    - /dev/hugepages
    debugfs                0       0         0    - /sys/kernel/debug
    mqueue                 0       0         0    - /dev/mqueue
    tmpfs             101544       0    101544   0% /run/user/0
    binfmt_misc            0       0         0    - /proc/sys/fs/binfmt_misc
```