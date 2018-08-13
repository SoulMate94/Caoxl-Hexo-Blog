---
title: Linux 系统优化小建议
date: 2018-07-16 09:37:05
categories: Linux
tags: [Linux]
---

> 关于Linux系统优化的几个小建议

<!-- more -->

# `Linux` 系统优化

1. 登录系统: 不使用`root`登录, 通过`sudo`授权管理, 使用普遍用户登录

2. 禁止`SSH`进程: 更改默认的远程连接`SSH`服务及禁止`root`远程连接

3. 时间同步: 定时自动更新服务器时间

4. 配置`yum`更新源, 从国内更新下载更新`rpm`包

5. 关闭`selinux`及`iptabls` (`iptables` 工作场景如有 `wan ip`, 一般要打开, 高并发除外)

6. 调整文件描述符数量, 进程及文件的打开都会消耗文件描述符

7. 定时自动清理`/var/spool/clientmquene/目录垃圾文件`, 防止节点被占满 (`c6.4`默认没有`sendmial`, 因此可以不配)

8. 精简开机启动服务 (`crond`、 `sshd`、 `network`、`rsyslog`)

9. `Linux`内核参数优化 `/etc/sysctl.conf`, 执行`sysct-p`生效

10. 更改字符集, 支持中文, 但是还是建议使用英文, 防止乱码问题出现

11. 锁定关键系统文件 (`chattr +i /etc/passwd /etc/shadow /etc/group /etc/gshadow /etc/inittab 处理以上内容后，把chatter`改名，就更安全了。)

12. 清空 `/etc/issue`, 去除系统及内核版本登陆前的屏幕显示


# `Linux` 启动过程

> 描述`Linux`系统从开机到登陆界面的启动过程, 了解整个过程, 才知道在哪个地方着手优化

1. 开机`BIOS`自检，加载硬盘

2. 读取`MBR`, `MBR`引导

3. `grub`引导菜单(`Boot Loader`)

4. 加载内核`kernel`

5. 启动`init`进程，依据`inittab`文件设定运行级别

6. `init`进程，执行`rc.sysinit`文件

7. 启动内核模块, 执行不同级别的脚本程序

8. 执行`/etc/rc.d/rc.local`

9. 启动`mingetty`, 进入系统登录界面