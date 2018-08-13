---
title: Linux 命令 「halt」
date: 2018-06-28 14:25:30
categories: Linux cmd
tags: [Linux]
---

> **halt命令**用来关闭正在运行的`Linux`操作系统。`halt`命令会先检测系统的`runlevel`，
若`runlevel`为`0`或`6`，则关闭系统，否则即调用`shutdown`来关闭系统。

<!-- more -->

# 语法

```
    halt (选项)
```

# 选项

- `-d`:   不要在`wtmp`中记录
- `-f`:   不论目前的`runlevel`为何, 不调用`shudown`即强制关闭系统
- `-i`:   在`halt`之前,关闭全部的网络界面
- `-n`:   `halt`前,不用先执行`sync`
- `-p`:   `halt`之后,执行`poweroff`
- `-w`:   仅在`wtmp`中记录,而不实际结束系统

# 实例

```
    halt -p  // 关闭系统后关闭电源
    halt -d  // 关闭系统,但不留下记录
```
