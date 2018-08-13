---
title: Linux 命令 「dnf」
date: 2018-06-13 11:26:19
categories: Linux cmd
tags: [Linux]
---

> DNF D:掉你线 N:拿你钱 F:封你号, 你们就是那群穿着西装的死肥宅? 

额, 拉回主题~~~

> `DNF`是新一代的rpm软件包管理器。他首先出现在 Fedora 18 这个发行版中。而最近，它取代了yum，正式成为 Fedora 22 的包管理器。
DNF包管理器克服了YUM包管理器的一些瓶颈，提升了包括用户体验，内存占用，依赖分析，运行速度等多方面的内容。
DNF使用 RPM, libsolv 和 hawkey 库进行包管理操作。

<!-- more -->

# 安装 `DNF`

DNF 并未默认安装在 RHEL 或 CentOS 7系统中，但是 Fedora 22 已经默认使用 DNF .

- 查看系统版本: `cat /etc/redhat-release`

1. 为了安装 DNF ，您必须先安装并启用 epel-release 依赖。

```
    yum install epel-release
    
    // 或
    yum install epel-release -y
```

2. 使用 `epel-release` 依赖中的 YUM 命令来安装 DNF 包。在系统中执行以下命令：

```
    yum install dnf
```

## `No package dnf available. ?`

```
    wget http://springdale.math.ias.edu/data/puias/unsupported/7/x86_64/dnf-conf-0.6.4-2.sdl7.noarch.rpm
    wget http://springdale.math.ias.edu/data/puias/unsupported/7/x86_64//dnf-0.6.4-2.sdl7.noarch.rpm
    wget http://springdale.math.ias.edu/data/puias/unsupported/7/x86_64/python-dnf-0.6.4-2.sdl7.noarch.rpm
    yum install python-dnf-0.6.4-2.sdl7.noarch.rpm  dnf-0.6.4-2.sdl7.noarch.rpm dnf-conf-0.6.4-2.sdl7.noarch.rpm
```

# 实例

- 查看`DNF`包管理器版本

```
    dnf -version
```

- 查看系统中可用的`DNF`软件库

```
    dnf repolist
```

- 查看系统中可用和不可用的所有`DNF`软件库

```
    dnf repolist all
```

- 列出所有`RPM`包

```
    dnf list
```

- 列出所有安装了的`RPM`包

```
    dnf list installed
```

- 列出所有可供安装的`RPM`包

```
    dnf list avaiable
```

- 搜索软件库的`RPM`包

```
    dnf search nano // 软件的部分名称
```

- 查看某一文件的提供者

```
    dnf provides /bin/bash
```

- 查看软件包详情

```
    dnf info nano
```

- 安装软件包

```
    dnf install nano
```

- 升级软件包

```
    dnf update system
```

- 检查系统软件包的更新

```
    dnf check-update
```

- 升级所有系统软件包

```
    dnf update
    
    或
    dnf upgrade
```

- 删除软件包

```
    dnf remove nano
    
    或
    dnf erase nano
```

- 删除无用孤立的软件包

```
    dnf autoremove
```

- 删除缓存的无用软件包

```
    dnf clean all
```

- 获取有关某条命令的使用帮助

```
    dnf help clean
```

- 查看所有的`dnf`命令及其用途

```
    dnf help
```

- 查看`dnf`命令的执行历史

```
    dnf history
```

- 查看所有的软件包组

```
    dnf grouplist
```

- 安装一个软件包组

```
    dnf groupinstall "Educational Software"
```

- 升级一个软件包组中的软件包

```
    dnf groupupdate "Educational Software"
```

- 删除一个软件包组

```
    dnf groupremove "Educational Software"
```

- 从特定的软件包库安装特定的软件

```
    dnf –enablerepo=epel install phpmyadmin
```

- 更新软件包到最新的稳定发行版

```
    dnf distro-sync
```

- 重新安装特定软件包

```
    dnf reinstall nano
```

- 回滚某个特定软件的版本

```
    dnf downgrade acpid
    // 该命令用于降低特定软件包的版本
```

样例输出：

```
    [root@izj6c6djex81rijczh0t8yz ~]# dnf downgrade acpid
    Using metadata from Wed Jun 13 11:36:20 2018
    No match for available package: acpid-2.0.19-9.el7.x86_64
    Error: Nothing to do.
```

> 原作者注：在执行这条命令的时候， `DNF` 并没有按照我期望的那样降级指定的软件（`“acpid”`）。该问题已经上报。

# 总结

`DNF` 包管理器作为 `YUM` 包管理器的升级替代品，它能自动完成更多的操作。
但在我看来，正因如此，所以 `DNF` 包管理器不会太受那些经验老道的 `Linux` 系统管理者的欢迎。举例如下：

1. 在 `DNF` 中没有 `–skip-broken` 命令，并且没有替代命令供选择。
2. 在 `DNF` 中没有判断哪个包提供了指定依赖的 `resolvedep` 命令。
3. 在 `DNF` 中没有用来列出某个软件依赖包的 `deplist` 命令。
4. 当你在 `DNF` 中排除了某个软件库，那么该操作将会影响到你之后所有的操作，
不像在 `YUM` 下那样，你的排除操作只会咋升级和安装软件时才起作用。

****
