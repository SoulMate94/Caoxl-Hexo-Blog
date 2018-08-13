---
title: Linux 命令 「systemctl」
date: 2018-06-07 10:54:23
categories: Linux cmd
tags: [Linux]
---

# `systemcl`

`systemctl`命令是系统服务管理器指令，
它实际上将 `service` 和 `chkconfig` 这两个命令组合到一起。

<!-- more -->


# 命令

- 使某服务自动启动: `systemctl enable httpd.service`
- 使某服务不自动启动: `systemctl disable httpd.service`
- 检查服务状态: 
  - `systemctl status httpd.service (服务详细信息)`
  - `systemctl is-active httpd.service (仅显示是否Active)`
- 显示所有已启动的服务: `systemctl list-units --type=service`
- 启动某服务: `systemctl start httpd.service`
- 停止某服务: `systemctl stop httpd.service`
- 重启某服务: `systemctl restart httpd.service`


# 实例

- 1.启动`nfs`服务

```$xslt
    systemctl start nfs-server.service
```

- 2.设置开机自启动

```$xslt
    systemctl enable nfs-server.service
```

- 3.停止开机启动

```$xslt
    systemctl disable nfs-server.service
```

- 4.查看服务当前状态

```$xslt
    systemctl status nfs-server.service
```

- 5.重新启动某服务

```$xslt
    systemctl restart nfs-server.service
```

- 6.查看所有已启动的服务

```$xslt
    systemctl list -units --type=service
```

- 开启防火墙22端口

```$xslt
    iptables -I INPUT -p tcp --dport 22 -j accept
```

> 如果仍然有问题，就可能是SELinux导致的
> 
> 关闭SElinux：
> 
> 修改/etc/selinux/config文件中的SELINUX=””为disabled，然后重启。
 
- 彻底关闭防火墙：

```$xslt
    sudo systemctl status firewalld.service
    sudo systemctl stop firewalld.service
    sudo systemctl disable firewalld.service
```


