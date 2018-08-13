---
title: Linux 命令 「whereis」
date: 2018-07-02 10:07:59
categories: Linux cmd
tags: [Linux]
---

> **whereis命令**用来定位指令的二进制程序、源代码文件和`man`手册页等相关文件的路径。

<!-- more -->

`whereis命令`只能用于程序名的搜索，
而且只搜索`二进制文件（参数-b）`、`man说明文件（参数-m）`和`源代码文件（参数-s）`。如果省略参数，则返回所有信息。

> 和`find`相比，`whereis`查找的速度非常快，这是因为`linux`系统会将系统内的所有文件都记录在一个数据库文件中，
当使用`whereis`和下面即将介绍的`locate`时，会从数据库中查找数据，而不是像find命令那样，通 过遍历硬盘来查找，效率自然会很高。 
但是该数据库文件并不是实时更新，默认情况下时一星期更新一次，
因此，我们在用whereis和locate 查找文件时，有时会找到已经被删除的数据，
或者刚刚建立文件，却无法查找到，原因就是因为数据库文件没有被更新。


# 命令选项

- `-b`:   只查找二进制文件
- `-B`<目录>:   只在设置的目录下查找二进制文件
- `-f`:   不显示文件名前的路径名称
- `-m`:   只查找说明文件
- `-M`<目录>:   只在设置的目录下查找说明文件
- `-s`:   只查找原始代码文件
- `-S`<目录>:   只在设置的目录下查找原始代码文件
- `-u`:   查找不包含指定类型的文件

# 命令参数

- 指令名:  要查找的二进制程序,源文件和`man`手册页的指令名

# 实例

- 将相关的文件都查找出来

```
    [root@izj6c6djex81rijczh0t8yz ~]# whereis tomcat
    tomcat:
    [root@izj6c6djex81rijczh0t8yz ~]# whereis svn
    svn:
    [root@izj6c6djex81rijczh0t8yz ~]# whereis git
    git: /usr/bin/git /usr/share/man/man1/git.1.gz
```

说明：`tomcat`、`svn`没安装，找不出来，`git`安装找出了很多相关文件

- 只将二进制文件查找出来 

```
    # 只查找二进制文件
    [root@izj6c6djex81rijczh0t8yz ~]# whereis -b git
    git: /usr/bin/git
    
    # 只查找说明文件
    [root@izj6c6djex81rijczh0t8yz ~]# whereis -m git
    git: /usr/share/man/man1/git.1.gz
    
    # 只查找`source`源文件
    [root@izj6c6djex81rijczh0t8yz ~]# whereis -s git
    git:
```