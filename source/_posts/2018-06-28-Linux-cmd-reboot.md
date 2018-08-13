---
title: Linux 命令 「reboot」
date: 2018-06-28 14:34:33
categories: Linux cmd
tags: [Linux]
---

> **reboot命令**用来重新启动正在运行的Linux操作系统。

<!-- more -->

# 语法

```
    reboot (选项)
```

# 选项

- `-d`:   重新开机时不把数据写入记录文件`/var/tmp/wtmp/.本参数具有`-n`参数效果
- `-f`:   强制重新开机,不调用`shutdown`指令的功能
- `-i`:   在重开机之前,先关闭所有网络界面
- `-n`:   重开机之前不检查师傅有未结束的程序
- `-w`:   仅做测试,并不真正的将系统重新开机,只会把重开机的数据写入`/var/log`目录下的`wtmp`记录文件。


# 实例

```
    reboot  // 重开机
    reboot -w // 做个重开机的模拟
```