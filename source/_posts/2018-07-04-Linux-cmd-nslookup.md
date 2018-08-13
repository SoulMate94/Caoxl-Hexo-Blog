---
title: Linux 命令 「nslookup」
date: 2018-07-04 14:10:41
categories: Linux cmd
tags: [Linux]
---

> **nslookup命令**是常用域名查询工具，就是查`DNS`信息用的命令。

<!-- more -->

`nslookup`有两种工作模式，即“交互模式”和“非交互模式”。

在“交互模式”下，用户可以向域名服务器查询各类主机、域名的信息，或者输出域名中的主机列表。
而在“非交互模式”下，用户可以针对一个主机或域名仅仅获取特定的名称或所需信息。

# 命令选项

- `-sil:`   不显示任何警告信息

# 命令参数

- 域名:   指定要查询域名

# 实例

```php
    [root@izj6c6djex81rijczh0t8yz ~]# nslookup
    > www.caoxl.com     
    Server:		100.100.2.136
    Address:	100.100.2.136#53
    
    Non-authoritative answer:
    Name:	www.caoxl.com
    Address: 47.91.221.85
```
