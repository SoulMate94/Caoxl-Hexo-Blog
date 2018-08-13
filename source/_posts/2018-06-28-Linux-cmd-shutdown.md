---
title: Linux 命令 「shutdown」
date: 2018-06-28 14:42:31
categories: Linux cmd
tags: [Linux]
---

> **shutdown命令**用来系统关机命令。shutdown指令可以关闭所有程序，并依用户的需要，进行重新开机或关机的动作。

<!-- more -->

# 语法

```
    shutdown (选项)(参数)
```

# 选项

- `-c`:   当执行一个定时指令时,只要按`+`键就可以中断关机的命令
- `-f`:   重新启动不执行`fsck`
- `-F`:   重新启动时执行`fsck`
- `-h`:   将系统关机
- `-k`:   只是送出信息给所有用户,但不会实际关机
- `-n`:   不调用`init`程序进行关机,而由`shutdown`自己进行
- `-r`:   `shutdown`之后重新启动
- `-t<秒数>`:   送出警告信息和删除信息之间要延迟多少秒

# 参数

- [时间]: 设置多久时间后执行`shutdown`指令
- [警告信息]: 要传递给所有登入用户的信息

# 实例

- 指定现在立即关机

```
    shutdown -h now
```

- 指定5分钟后关机, 同时发出警告信息给登入用户

```
    shutdown +5 "System will shutdown after 5 minutes"
```