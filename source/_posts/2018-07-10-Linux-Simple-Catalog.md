---
title: Linux 系统目录 (简易版)
date: 2018-07-10 16:33:29
categories: Linux
tags: [Linux]
---

> 曾经做了一个详细的目录说明 [Linux 系统中的目录](http://blog.caoxl.com/2018/02/01/Linux-Catalog/)

<!-- more -->

# 目录结构

- 1. `root` / `~`

超级用户的家目录

```
    cd /root 相当于 cd ~
```

- 2. `home`

普通用户的家目录

- 3. `bin`

所有用户都可以执行的命令

- 4. `sbin`

只有`root`用户才可以执行的命令

- 5. `etc`

软件的配置文件

```
    /etc/init.d   # 系统服务文件夹
```

- 6. `usr`

自定义安装的软件, 执行程序, 帮助手册


- 7. `var`

日志, 缓存

- 8. `boot`

启动文件, 包括`linux`内核, `init`进程镜像文件

- 9. `tmp`

临时目录

- 10. `dev`

硬盘设备文件

- 11. `media`

挂载光盘用的

- 12. `mnt`

空目录

- 13. `proc`

内存镜像文件

- 14. `dev`

存放设备节点

- 15. `lib`

存放核心系统程序需要的库文件

- 16. `lib64`

如果你使用的是`X86_64`的`Linux`系统, 那可能会有`/lib64/`目录产生 

- 17. `opt`

用来安装“可选的”软件,

- 18. `srv`

`service`的缩写, 存储:网络服务启动后，这些服务所需要取用的数据目录,一般是空目录

- 19. `lost+found`

恢复一个损坏的文件系统时，会用到这个目录, 一般是空目录

- 20. `sys`

存储虚拟文件系统

- 21. `run`

某些程序或者服务启动后，会将他们的 `PID` 放置在这个目录下。

# 参考

- [linux各文件夹的作用](https://www.cnblogs.com/amboyna/archive/2008/02/16/1070474.html)
- [linux 目录结构及其含义](https://www.cnblogs.com/zhyryxz/archive/2012/05/03/2480242.html)