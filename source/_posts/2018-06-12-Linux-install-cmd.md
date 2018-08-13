---
title: Linux 安装命令集合
date: 2018-06-12 10:31:06
categories: Linux
tags: Linux
---

> 整理Linux下的不同系统的下载安装命令

<!-- more -->

# `yum`

**yum命令**是在Fedora和RedHat以及SUSE中基于`rpm`的软件包管理器，
它可以使系统管理人员交互和自动化地更细与管理RPM软件包，能够从指定的服务器自动下载RPM包并且安装，
可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。

`yum`提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

## 语法

```
    yum (选项)(参数)
```

## 选项

- `-h`: 显示帮助信息
- `-y`: 对所有的提问都回答"yes"
- `-c`: 指定配置文件
- `-q`: 安静模式
- `-v`: 详细模式
- `-d`: 设置调试等级(0-10)
- `-e`: 设置错误等级(0-10)
- `-R`: 设置yum处理一个命令的最大等待时间
- `-C`: 完成从缓存中运行, 而不去下载或更新任何文件

## 参数

- `install`:  安装rpm软件包
- `update`:   更新rpm软件包
- `check-update`: 检查是否有可用的更新rpm软件包
- `remove`:   删除指定的rpm软件包
- `list`:     显示软件包的信息
- `search`:   检查软件包的信息
- `info`:     显示指定的rpm软件包的描述信息和概要信息
- `clean`:    清理yum过期的缓存
- `shell`:    进入yum的shell提示符
- `resolvedep`: 显示rpm软件包的依赖关系
- `localinstall`: 安装本地的rpm软件包
- `localupdate`:  显示本地rpm软件包进行更新
- `deplist`:  显示rpm软件包的所有依赖关系

## 实例

### 常用命令

- 自动搜索最快镜像插件

```
    yum install yum-fastestmirror
```

- 安装yum图形窗口插件

```
    yum install yumex
```

- 查看可能批量安装的列表

```
    yum grouplist
```

### 安装

```
    yum install             #全部安装
    yum install package1    #安装指定的安装包package1
    yum groupinsatll group1 #安装程序组group1
```

### 更新和升级

```
    yum update              #全部更新
    yum update package1     #更新指定程序包package1
    yum check-update        #检查可更新的程序
    yum upgrade package1    #升级指定程序包package1
    yum groupupdate group1  #升级程序组group1
```

### 查找和显示

```
    yum info package1       #显示安装包信息package1
    yum list                #显示所有已经安装和可以安装的程序包
    yum list package1       #显示指定程序包安装情况package1
    yum groupinfo group1    #显示程序组group1信息yum search string 根据关键字string查找安装包
```

### 删除程序

```
    yum remove &#124; erase package1    #删除程序包package1
    yum groupremove group               #删除程序组group1
    yum deplist package1                #查看程序package1依赖情况
```

### 清除缓存

```
    yum clean packages      #清除缓存目录下的软件包
    yum clean headers       #清除缓存目录下的headers
    yum clean oldheaders    #清除缓存目录下旧的headers
```

# `apt-get`

**apt-get命令**是`Debian Linux`发行版中的APT软件包管理工具。所有基于`Debian`的发行都使用这个包管理系统。
`deb`包可以把一个应用的文件包在一起，大体就如同`Windows`上的安装文件。

## 语法

```
    apt-get(选项)(参数)
```

## 选项

- `-c`:   指定配置文件

## 参数

- 管理指令: 对APT软件包的管理操作
- 软件包:  指定要操纵的软件包

## 实例

- 安装

```
    apt-get install packagename
```

- 卸载一个已安装的软件包（保留配置文件）：

```
    apt-get remove packagename
```

- 卸载一个已安装的软件包（删除配置文件）：

```
    apt-get -purge remove packagename
```

- 删除你已经删掉的软件：

```
    apt-get autoclean apt
```

- 把安装的软件的备份也删除

```
    apt-get clean
```

- 更新所有已安装的软件包：

```
    apt-get upgrade
```

- 将系统升级到新版本：

```
    apt-get dist-upgrade
```

# `rpm`

**rpm命令**是RPM软件包的管理工具

## 语法

```
    rpm (选项)(参数)
```

## 选项

- `-a`:   查询所有套件
- `-b`<完成阶段><套件挡>+或-t<完成阶段><套件档>+:    设置包装 套件的完成阶段,并指定套件档的文件名称
- `-c`:   只列出组态配置文件,本参数需配合"-l"参数使用
- `-d`:   只列出文本文件,本参数需配合"-l"参数使用
- `-e`<套件档>或--erase<套件档>: 删除指定的套件
- `-f`<文件>+:   查询拥有指定文件的套件
- `-h/--hash`:    套件安装时列出标记
- `-i`:   显示套件的相关信息
- `-l`:   显示套件的文件列表
- `-p`<套件档>+:  查询指定的文件列表
- `-q`:   使用询问模式,当遇到任何问题是,rpm指令会先询问用户
- `-R`:   显示套件的关联性信息
- `-s`:   显示文件状态,本参数需要配置"-l"参数使用
- `-U`<套件档>或--upgrade<套件档>：升级指定的套件档
- `-v`:   显示指令执行过程
- `-vv`:  详细显示指令执行过程,便于排错

## 参数

- 软件包:  指定要操纵的rpm软件包

## 实例

```
    rpm -ivh your-package.rpm
```

### 如何安装.src.rpm软件包

有些软件包是以`.src.rpm`结尾的，这类软件包是包含了源代码的`rpm`包,
在安装时需要进行编译。这类软件包有两种安装方法：

- 方法一:

```
    rpm -i your-package.src.rpm
    cd /usr/src/redhat/SPECS
    rpmbuild -bp your-package.specs             #一个和你的软件包同名的specs文件
    cd /usr/src/redhat/BUILD/your-package/      #一个和你的软件包同名的目录
    ./configure                                 #这一步和编译普通的源码软件一样，可以加上参数
    make
    make install
```

- 方法二:

```
    rpm -i you-package.src.rpm
    cd /usr/src/redhat/SPECS
```

前两步和方法一相同

```
    rpmbuild -bb your-package.specs       #一个和你的软件包同名的specs文件
    
    执行rpm -i new-package.rpm即可安装完成。
```


### 如何卸载rpm软件包

使用命令rpm -e包名，包名可以包含版本号等信息，
但是不可以有后缀.rpm，比如卸载软件包proftpd-1.2.8-1，可以使用下列格式：

```
    rpm -e proftpd-1.2.8-1
    rpm -e proftpd-1.2.8
    rpm -e proftpd-
    rpm -e proftpd
```

不可以是下列格式：


```
    rpm -e proftpd-1.2.8-1.i386.rpm
    rpm -e proftpd-1.2.8-1.i386
    rpm -e proftpd-1.2
    rpm -e proftpd-1
```

### 如何不安装但是获取rpm包中的文件

使用工具`rpm2cpio`和`cpio`

```
    rpm2cpio xxx.rpm | cpio -vi
    rpm2cpio xxx.rpm | cpio -idmv
    rpm2cpio xxx.rpm | cpio --extract --make-directories
```

### 如何查看与rpm包相关的文件和其他信息

- 我的系统中安装了那些rpm软件包。

```
    rpm -qa 将列出所有安装过的包
```

- 如果要查找所有安装过的包含某个字符串sql的软件包

```
    rpm -qa | grep sql
```

- 如何获得某个软件包的文件全名。

```
    rpm -q mysql
```


# `pear/pecl`

## `PEAR`

- go-pear会同时安装 pear 和 pecl 命令

```
    #这是一个安装 pear 的 php 发行包文件
    wget http://pear.php.net/go-pear.phar
    #执行安装
    php go-pear.phar
```

- pear 升级或更新源

```
    #如果想升级到最新版本
    pear upgrade --force PEAR
    #更新下仓库
    pecl channel-update pecl.php.net
```

- pear 安装扩展工具

```
    pear install DB
```

## `PECL`

`PHP Extension Community Library` php 的 `C` 扩展仓库，即 php 的 `so` 格式的扩展

因为是 C 所以得装个编译器

```
    yum groupinstall "Development tools"
    yum -y install gcc gcc-c++ make cmake automake autoconf
```

安装redis扩展

```
    pecl info redis
    pecl install redis
    pecl uninstall redis
    
    #也可以使用安装包
    wget http://pecl.php.net/get/redis-4.0.0.tgz
    pecl install redis-4.0.0.tgz
    
    // 就会生成 redis.so 文件，加入到 php.ini 中即可
```

# `dpkg`

**dpkg命令**是Debian Linux系统用来安装、创建和管理软件包的实用工具。

## 选项/参数

- `-i`:   安装软件包
- `-r`:   删除软件包
- `-P`:   删除软件包的同时删除其配置文件
- `-L`:   显示与软件包关联的文件
- `-l`:   显示已安装软件包列表
- `--unpack`: 解开软件包
- `-c`:   显示软件包内文件列表
- `--configure`: 配置软件包

- 参数

Deb软件包: 指定要操纵的`.deb`软件包

## 实例

```
    dpkg -i package.deb     #安装包
    dpkg -r package         #删除包
    dpkg -P package         #删除包(包括配置文件)
    dpkg -L package         #列出与该包关联的文件
    dpkg -l package         #显示该包的版本
    dpkg --unpack package.deb #解开deb包的内容
    dpkg -S keyword         #搜索所属的包内容
    dpkg -l                 #列出deb包的内容
    dpkg -c package.deb     #列出deb包的内容
    dpkg --configure package  #配置包
```

# `tar.gz`源码包安装

- 1. 找到相应的软件包，比如soft.tar.gz，下载到本机某个目录

- 2. 打开一个终端，`su root`

- 3. `cd soft.tar.gz`所在的目录；

- 4. `tar -xzvf soft.tar.gz` //一般会生成一个soft目录

- 5. `cd soft`

- 6. `./configure`

- 7. `make && make install`

