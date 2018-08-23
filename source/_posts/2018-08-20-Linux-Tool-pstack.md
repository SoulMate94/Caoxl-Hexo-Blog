---
title: Linux pstack 跟踪进程栈
date: 2018-08-20 10:09:34
categories: Linux tool
tags: [Linux, pstack]
---

> 此命令可显示每个进程的栈跟踪。`pstack` 命令必须由相应进程的属主或 `root` 运行。可以使用 `pstack` 来确定进程挂起的位置。此命令允许使用的唯一选项是要检查的进程的 `PID`

<!-- more -->

# 应用场景

这个命令在排查进程问题时非常有用，比如我们发现一个服务一直处于work状态（如假死状态，好似死循环），使用这个命令就能轻松定位问题所在；可以在一段时间内，多执行几次pstack，若发现代码栈总是停在同一个位置，那个位置就需要重点关注，很可能就是出问题的地方

# 示例: 查看bash程序进程栈

```
    [root@caoxl ~]# ps -ef | grep bash
    root     31039 31037  0 09:42 pts/0    00:00:00 -bash
    root     31229 31039  0 10:11 pts/0    00:00:00 grep --color=auto bash
    [root@caoxl ~]# pstack 31039
    #0  0x00007fe8aa1e817c in waitpid () from /usr/lib64/libc.so.6
    #1  0x0000000000440b24 in waitchld.isra.10 ()
    #2  0x0000000000441ddc in wait_for ()
    #3  0x0000000000433aae in execute_command_internal ()
    #4  0x0000000000433cce in execute_command ()
    #5  0x000000000041e305 in reader_loop ()
    #6  0x000000000041c96e in main ()
```

# 安装pstack

```
    yum install gdb  // 只要安装gdb,就会把pstack也一并安装成功
    
    [root@caoxl ~]# which pstack
    /usr/bin/pstack
```


# 参考

- [pstack 跟踪进程栈](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/pstack.html)
