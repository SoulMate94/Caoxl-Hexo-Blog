---
title: Linux 命令 「find」
date: 2018-07-25 17:04:10
categories: Linux cmd
tags: [Linux]
---

> **find**是一个基于查找的功能非常强大的命令，相对而言，它的使用也相对较为复杂，参数也比较多

<!-- more -->

# 语法

```
    find [PATH] [option] [action]
```

# 参数

## 与时间有关的参数

- `mtime n` : n为数字，意思为在n天之前的“一天内”被更改过的文件
- `mtime +n` : 列出在n天之前（不含n天本身）被更改过的文件名
- `mtime -n` : 列出在n天之内（含n天本身）被更改过的文件名
- `newer file` : 列出比file还要新的文件名

**例如:**

```
    find /root -mtime 0  # 在/root目录下查找今天之内有改动的文件
```



## 与用户或用户组名有关的参数

- `user name` : 列出文件所有者为`name`的文件
- `group name` : 列出文件所属用户组为`name`的文件
- `uid n` : 列出文件所有者为用户`ID`为`n`的文件
- `gid n` : 列出文件所属用户组为用户组`ID`为`n`的文件

**例如:**

```
    find /home/caoxl -user caoxl # 在目录/home/caoxl 中找出所有者为caoxl的文件
```

## 与文件权限及名称有关的参数

- `name filename` ：找出文件名为filename的文件
- `size [+-]SIZE` ：找出比SIZE还要大（+）或小（-）的文件
- `tpye TYPE` ：查找文件的类型为TYPE的文件
  - TYPE的值主要有：一般文件（f)、设备文件（b、c）、目录（d）、连接文件（l）、socket（s）、FIFO管道文件（p）；
- `perm mode` ：查找文件权限刚好等于mode的文件，mode用数字表示，如0755；
- `perm -mode` ：查找文件权限必须要全部包括mode权限的文件，mode用数字表示
- `perm +mode` ：查找文件权限包含任一mode的权限的文件，mode用数字表示

**例如:**

```
    find / -name passwd  # 查找文件名为passwd的文件
    find . -perm 0755    # 查找当前目录中文件权限为0755的文件
    find . -size +12k    # 查找当前目录中大于12KB的文件, 注意c表示type
```
