---
title: Linux 命令 「uname」
date: 2018-06-28 14:07:19
categories: Linux cmd
tags: [Linux]
---

> **uname命令**用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。

<!-- more -->

# 命令参数

- `-a`/`--all`: 显示全部的信息
- `-m`/`--machine`: 显示电脑类型
- `-n`/`-nodename`: 显示在网络上的主机名称
- `-r`/`--release`: 显示操作系统的发行编号
- `-s`/`--sysname`: 显示操作系统名称
- `-v`:   显示操作系统的版本
- `-p`/`--processor`:   输出处理器类型或"unknown"
- `-i`/`--operating-system`: 输出操作系统名称
- `--help`:   显示帮助
- `--version`: 显示版本信息

# 实例

- `uname`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname
    Linux
```

- `uname -a`

```
   [root@izj6c6djex81rijczh0t8yz test]# uname -a
   Linux izj6c6djex81rijczh0t8yz 3.10.0-862.3.2.el7.x86_64 #1 SMP Mon May 21 23:36:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux 
```

- `uname -m`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -m
    x86_64
```

- `uname -n`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -n
    izj6c6djex81rijczh0t8yz
```

- `uname -r`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -r
    3.10.0-862.3.2.el7.x86_64
```

- `uname -s`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -s
    Linux
```

- `uname -v`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -v
    #1 SMP Mon May 21 23:36:36 UTC 2018
```

- `uname -p`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -p
    x86_64
```

- `uname -i`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -i
    x86_64
```

- `uname -o`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname -o
    GNU/Linux
```

- `uname --version`

```
    [root@izj6c6djex81rijczh0t8yz test]# uname --version
    uname (GNU coreutils) 8.22
    Copyright (C) 2013 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    
    Written by David MacKenzie.
```