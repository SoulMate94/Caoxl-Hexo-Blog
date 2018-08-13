---
title: Linux 命令 「kill」
date: 2018-06-12 09:17:26
categories: Linux cmd
tags: [Linux]
---

> **kill命令**用来删除执行中的程序或工作。kill可将指定的信息送至程序。
预设的信息为SIGTERM(15),可将指定程序终止。若仍无法终止该程序，可使用SIGKILL(9)信息尝试强制删除程序。
程序或工作的编号可利用`ps`指令或`job`指令查看。

<!-- more -->

# 命令参数

- `-a`: 当处理当前进程时, 不限制命令名和进程名的对应关系
- `-l<信息编号>`: 若不加<信息编号>选项,则-l参数惠列出全部的信息名称
- `-p`: 指定kill命令只打印相关进程的进程号,而不发送任何信号
- `-s<信息名称或编号>`: 指定要送出的信息
- `-u`: 指定用户

## 所有信号名称

- `kill -l`

```
    [root@izj6c6djex81rijczh0t8yz ~]# kill -l
     1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
     6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
    11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
    16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
    21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
    26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
    31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
    38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
    43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
    48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
    53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
    58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
    63) SIGRTMAX-1	64) SIGRTMAX	
```

只有第9种信号(SIGKILL)才可以无条件终止进程，其他信号进程都有权利忽略，下面是常用的信号：

```
    HUP     1       终端断线
    INT     2       中断(同Ctrl + C)
    QUIT    3       退出(同Ctrl + \)
    TERM    15      终止
    KILL    9       强制终止
    CONT    18      继续(与STOP相反, fg/bg命令)
    STOP    19      暂停(同Ctrl + Z)
```

# 使用

先用`ps`查找进程，然后用`kill`杀掉：

```
    ps -ef | grep vim
    kill PID
```

**注意:**

>   使用信号 `15` 是安全的，而信号 `9` 则是处理异常进程的最后手段，请勿滥用。


# `pkill`

**pkill命令**可以按照进程名杀死进程。
`pkill`和`killall`应用方法差不多，也是直接杀死运行中的程序；如果您想杀掉单个进程，请用kill来杀掉。

## 命令参数

- `-o`:   仅向找到的最小(起始)进程号发送信号
- `-n`:   仅向找到的最大(结束)进程号发送信号
- `-p`:   指定父进程号发送信号
- `-g`:   指定进程组

进程名称：指定要查找的进程名称，同时也支持类似`grep`指令中的匹配模式。

## 实例

```
    grep -l gaim
    2979 gaim
    
    pkill gaim
```

也就是说：`kill`对应的是`PID`，`pkill`对应的是`command`。


# `killall`

**killall命令**使用进程的名称来杀死进程，使用此指令可以杀死一组同名进程

## 命令参数

- `-e`:   对长名称进行精确匹配
- `-I`:   忽略大小写的不同
- `-p`:   杀死进程所属的进程组
- `-i`:   交互式杀死进程,杀死进程前需要进行确认
- `-l`:   打印所有已知信号列表
- `-q`:   如果没有进程被杀死,则不输出任何信息
- `-r`:   使用正则表达式匹配要杀死的进程名称
- `-s`:   用指定的进程号代替默认信号"SIGTERM"
- `-u`:   杀死指定用户的进程
- `-Z`:   只杀死拥有`scontext`的进程
- `-v`:   报告信息是否成功发送
- `--help`: 显示帮助信息

## 实例

- 杀死所有同名进程

```
    killall vi
```
