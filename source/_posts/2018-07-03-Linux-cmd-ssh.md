---
title: Linux 命令 「ssh」
date: 2018-07-03 09:55:58
categories: Linux cmd
tags: [Linux]
---

> **ssh命令**是`openssh`套件中的客户端连接工具，可以给予ssh加密协议实现安全的远程登录服务器。

<!-- more -->

# 语法

```
    ssh (选项)(参数)
```

# 选项

- `-1`:   强制使用ssh协议版本1;
- `-2`:   强制使用ssh协议版本2;
- `-4`:   强制使用IPv4地址;
- `-6`:   强制使用IPv6地址;
- `-A`:   开启认证代理连接转发功能;
- `-a`:   关闭认证代理连接转发功能;
- `-b`:   使用本机指定地址作为对应连接的源ip地址;
- `-C`:   请求压缩所有数据;
- `-F`:   指定ss指令的配置文件;
- `-f`:   后台执行ssh指令;
- `-g`:   允许远程主机连接主机的转发端口;
- `-i`:   指定身份文件;
- `-l`:   指定连接远程服务器登录用户名;
- `-N`:   不执行远程指令;
- `-o`:   指定配置选项;
- `-p`:   指定远程服务器上的端口;
- `-q`:   静默模式;
- `-X`:   关闭X11转发功能;
- `-y`:   开启信任X11转发功能;

# 参数

- 远程主机: 指定要连接的远程`ssh`服务器
- 指令:   要在远程`ssh`服务器上执行的指令

# 实例

- `ssh`登录远程服务器

```
    ssh user@host -p port
    
    例:
    ssh caoxl@127.0.0.1 -p 22
```