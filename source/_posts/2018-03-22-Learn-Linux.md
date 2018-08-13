---
title: Linux 基础
date: 2018-03-22 16:47:18
categories: Linux
tags: Linux
---


一个PHP瓜皮如何学习Linux?

<!-- more -->

# Linux 关机命令

正确的关机流程为：sync > shutdown > reboot > halt

> - sync 将数据由内存同步到硬盘中。
> - shutdown 机指令，你可以man shutdown 来看一下帮助文档。例如你可以运行如下命令关机
> - shutdown -h 10 "this server will shutdown after 10 mins"  这个命令告诉大家，计算机将在10分钟后关机，并且会显示在登陆用户的当前屏幕中。
> - Shutdown -h now  立马关机
> - Shutdown -h 20:35  系统会在今天20:25关机
> - Shutdown -h +10  十分钟后关机
> - Shutdown -r now  系统立马重启
> - Shutdown -r +10  系统十分钟后重启
> - reboot  就是重启，等同于 shutdown –r now
> - halt  关闭系统，等同于shutdown –h now 和 poweroff

最后总结一下，不管是重启系统还是关闭系统，首先要运行 sync 命令，把内存中的数据写到磁盘中。


# 目录结构

- `/bin`: bin是Binary的缩写, 这个目录存放着最经常使用的命令。
- `/boot`: 这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。
- `/dev`: dev是Device(设备)的缩写, 该目录下存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。
- `/etc`: 这个目录用来存放所有的系统管理所需要的配置文件和子目录。
- `/home`: 用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。
- `/lib`: 这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。
- `/lost+found`: 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。
- `/media`: linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。
- `mnt`: 系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。
- `/opt`:  这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。
- `/proc`: 这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。
这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件，比如可以通过下面的命令来屏蔽主机的ping命令，使别人无法ping你的机器

> echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

- `/root`: 该目录为系统管理员，也称作超级权限者的用户主目录。
- `/sbin`: s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。
- `/selinux`:  这个目录是Redhat/CentOS所特有的目录，Selinux是一个安全机制，类似于windows的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。
- `/srv`:  该目录存放一些服务启动之后需要提取的数据。
- `/sys`: 这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。

sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。
该文件系统是内核设备树的一个直观反映。

当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。

- `/tmp`: 这个目录是用来存放一些临时文件的。
- `/usr`: 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。
- `/usr/bin`: 系统用户使用的应用程序。
- `/usr/sbin`: 超级用户使用的比较高级的管理程序和系统守护程序。
- `/usr/src`: 内核源代码默认的放置目录。
- `/var`: 这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。


# Linux文件基本属性

```php
    [root@www /]# ls -l
    total 64
    dr-xr-xr-x   2 root root 4096 Dec 14  2012 bin
    dr-xr-xr-x   4 root root 4096 Apr 19  2012 boot
    ……
```

- 当为[ d ]则是目录
- 当为[ - ]则是文件；
- 若是[ l ]则表示为链接文档(link file)；
- 若是[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；
- 若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

接下来的字符中，以三个为一组，且均为『rwx』 的三个参数的组合。其中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。 要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。

![](http://www.runoob.com/wp-content/uploads/2014/06/363003_1227493859FdXT.png)

## 更改文件属性

### chgrp：更改文件属组

```php
    chgrp [-R] 属组名文件名
```

### chown：更改文件属主，也可以同时更改文件属组

```php
    chown [–R] 属主名 文件名
    chown [-R] 属主名：属组名 文件名
```


进入 /root 目录（~）将install.log的拥有者改为bin这个账号：

```php
    [root@www ~] cd ~
    [root@www ~]# chown bin install.log
    [root@www ~]# ls -l
    -rw-r--r--  1 bin  users 68495 Jun 25 08:53 install.log
```

将install.log的拥有者与群组改回为root：

```php
    [root@www ~]# chown root:root install.log
    [root@www ~]# ls -l
    -rw-r--r--  1 root root 68495 Jun 25 08:53 install.log
```

### chmod：更改文件9个属性

每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如当权限为： [-rwxrwx---] 分数则是：

- owner = rwx = 4+2+1 = 7
- group = rwx = 4+2+1 = 7
- others= --- = 0+0+0 = 0

变更权限的指令chmod的语法是这样的：

```php
    chmod [-R] xyz 文件或目录
```

- xyz : 就是刚刚提到的数字类型的权限属性，为 rwx 属性数值的相加。
- -R : 进行递归(recursive)的持续变更，亦即连同次目录下的所有文件都会变更


#### 符号类型改变文件权限

        u   +(加入)   r
chmod   g   -(除去)   w   文件或目录
        o   =(设定)   x      
        a
   

# Linux 文件与目录管理

## 处理目录的常用命令

- ls: 列出目录
- cd：切换目录
- pwd：显示目前的目录
- mkdir：创建一个新的目录
- rmdir：删除一个空的目录
- cp: 复制文件或目录
- rm: 移除文件或目录

## Linux 文件内容查看

- cat  由第一行开始显示文件内容
- tac  从最后一行开始显示，可以看出 tac 是 cat 的倒著写！
- nl   显示的时候，顺道输出行号！
- more 一页一页的显示文件内容
- less 与 more 类似，但是比 more 更好的是，他可以往前翻页！
- head 只看头几行
- tail 只看尾巴几行

# Linux 用户和用户组管理

## Linux系统用户账号的管理

### 添加新的用户账号

```php
    useradd 选项 用户名
```

### 删除帐号

```php
    userdel 选项 用户名
```

### 修改账号

```php
    usermod 选项 用户名
```

### 用户口令的管理

```php
    passwd 选项 用户名
```


## Linux系统用户组的管理

### 增加一个新的用户组

```php
    groupadd 用户组
```

### 删除一个用户组

```php
    groupdel 用户组
```

### 修改用户组

```php
    groupmod 选项 用户组
```


# Linux 磁盘管理

- df：列出文件系统的整体磁盘使用量
- du：检查磁盘空间使用量
- fdisk：用于磁盘分区

## df

> df [-ahikHTm] [目录或文件名]

- `-a` ：列出所有的文件系统，包括系统特有的 /proc 等文件系统；
- `-k` ：以 KBytes 的容量显示各文件系统；
- `-m` ：以 MBytes 的容量显示各文件系统；
- `-h` ：以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示；
- `-H` ：以 M=1000K 取代 M=1024K 的进位方式；
- `-T` ：显示文件系统类型, 连同该 partition 的 filesystem 名称 (例如 ext3) 也列出；
- `-i` ：不用硬盘容量，而以 inode 的数量来显示


## du

> du [-ahskm] 文件或目录名称

- `-a` ：列出所有的文件与目录容量，因为默认仅统计目录底下的文件量而已。
- `-h` ：以人们较易读的容量格式 (G/M) 显示；
- `-s` ：列出总量而已，而不列出每个各别的目录占用容量；
- `-S` ：不包括子目录下的总计，与 -s 有点差别。
- `-k` ：以 KBytes 列出容量显示；
- `-m` ：以 MBytes 列出容量显示


## fdisk

> fdisk [-l] 装置名称

- `-l` ：输出后面接的装置所有的分区内容。若仅有 fdisk -l 时， 则系统将会把整个系统内能够搜寻到的装置的分区均列出来。


## 磁盘检验

> fsck [-t 文件系统] [-ACay] 装置名称

- `-t` : 给定档案系统的型式，若在 /etc/fstab 中已有定义或 kernel 本身已支援的则不需加上此参数
- `-s` : 依序一个一个地执行 fsck 的指令来检查
- `-A` : 对/etc/fstab 中所有列出来的 分区（partition）做检查
- `-C` : 显示完整的检查进度
- `-d` : 打印出 e2fsck 的 debug 结果
- `-p` : 同时有 -A 条件时，同时有多个 fsck 的检查一起执行
- `-R` : 同时有 -A 条件时，省略 / 不检查
- `-V` : 详细显示模式
- `-a` : 如果检查有错则自动修复
- `-r` : 如果检查有错则由使用者回答是否修复
- `-y` : 选项指定检测每个文件是自动输入yes，在不确定那些是不正常的时候，可以执行 # fsck -y 全部检查修复。

## 磁盘挂载与卸除

> mount [-t 文件系统] [-L Label名] [-o 额外选项] [-n]  装置文件名  挂载点


# Yum 命令

## yum常用命令

- 1.列出所有可更新的软件清单命令：yum check-update
- 2.更新所有软件命令：yum update
- 3.仅安装指定的软件命令：yum install <package_name>
- 4.仅更新指定的软件命令：yum update <package_name>
- 5.列出所有可安裝的软件清单命令：yum list
- 6.删除软件包命令：yum remove <package_name>
- 7.查找软件包 命令：yum search <keyword>
- 8.清除缓存命令:
  - `yum clean packages`: 清除缓存目录下的软件包
  - `yum clean headers`: 清除缓存目录下的 headers
  - `yum clean oldheaders`: 清除缓存目录下旧的 headers
  - `yum clean, yum clean all (= yum clean packages; yum clean oldheaders)` :清除缓存目录下的软件包及旧的headers



# Shell 教程

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。


## 第一个shell脚本

```shell
    #!/bin/bash
    echo "Hello world!"
```

### 运行 Shell 脚本有两种方法：

- 1、作为可执行程序

```shell
    chmod +x ./test.sh  #使脚本具有执行权限
    ./test.sh  #执行脚本 
```

- 2、作为解释器参数

```shell
    /bin/sh test.sh
    /bin/php test.php
```

## Shell变量

### 使用变量

使用一个定义过的变量，只要在变量名前面加美元符号即可，如：

```shell
    your_name="qinjx"
    echo $your_name
    echo ${your_name}
```

变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

```shell
    for skill in Ada Coffe Action Java; do
        echo "I am good at ${skill}Script"
    done
```

### 只读变量

```shell
    #!/bin/bash
    myUrl="http://www.w3cschool.cc"
    readonly myUrl
    myUrl="http://www.runoob.com"
```


### 删除变量

```shell
    #!/bin/sh
    myUrl="http://www.runoob.com"
    unset myUrl
    echo $myUrl
```


## Shell 数组

```shell
    #!/bin/bash

    my_array=(A B "C" D)

    echo "第一个元素为: ${my_array[0]}"
    echo "第二个元素为: ${my_array[1]}"
    echo "第三个元素为: ${my_array[2]}"
    echo "第四个元素为: ${my_array[3]}"
```


## Shell 流程控制

### if else

```shell
    if condition
    then
        command1 
        command2
        ...
        commandN 
    fi
```

### if else-if else

```shell
    if condition1
    then
        command1
    elif condition2 
    then 
        command2
    else
        commandN
    fi
```

### for 循环

```php
    for var in item1 item2 ... itemN
    do
        command1
        command2
        ...
        commandN
    done
```

### while 语句

```shell
    while condition
    do
        command
    done
```

### 无限循环

```shell
    while :
    do
        command
    done

    // 或
    while true
    do
        command
    done

    // 或者
    for (( ; ; ))
```


### until 循环

```shell
    until condition
    do
        command
    done
```

### case

```shell
    case 值 in
    模式1)
        command1
        command2
        ...
        commandN
        ;;
    模式2）
        command1
        command2
        ...
        commandN
        ;;
    esac
```

### 跳出循环

#### break命令

```shell
    #!/bin/bash
    while :
    do
        echo -n "输入 1 到 5 之间的数字:"
        read aNum
        case $aNum in
            1|2|3|4|5) echo "你输入的数字为 $aNum!"
            ;;
            *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
                break
            ;;
        esac
    done
```

#### continue

```shell
    #!/bin/bash
    while :
    do
        echo -n "输入 1 到 5 之间的数字: "
        read aNum
        case $aNum in
            1|2|3|4|5) echo "你输入的数字为 $aNum!"
            ;;
            *) echo "你输入的数字不是 1 到 5 之间的!"
                continue
                echo "游戏结束"
            ;;
        esac
    done
```

# Linux 命令大全

- [命令大全](http://www.runoob.com/linux/linux-command-manual.html)