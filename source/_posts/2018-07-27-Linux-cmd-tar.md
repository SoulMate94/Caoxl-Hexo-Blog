---
title: Linux 命令 「tar」
date: 2018-07-27 09:30:19
categories: Linux
tags: [Linux, tar]
---

> 利用`tar`命令，可以把一大堆的文件和目录全部**打包**成一个文件

> 首先要弄清两个概念：**打包**和**压缩**。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件。

<!-- more -->

# 语法

```
    tar (选项) (参数)
```

# 选项

- `-A`/`--catenate`: 新增文件到已存在的备份文件
- `-B`: 设置区块大小
- `-c`/`--create`: 建立新的备份文件
- `-C`<目录>: 这个选项用在解压缩 若要在特定目录解压缩, 可以使用这个选项
- `-d`: 记录文件的差别
- `-x`/`--extract`/`--get`: 从备份文件中还原文件
- `-t`/`--list`: 列出备份文件的内容
- `-z`/`--gzip`/`--ungzip`: 通过gzip指令处理备份文件
- `-Z`/`--compress`/`--uncompress`: 通过compress指令处理备份文件
- `-f`<备份文件>/`--file`=<备份文件>: 指定备份文件
- `-v`/`--verbose`: 显示指令执行过程
- `-u`: 添加改变了和现有的文件到已经存在的压缩文件
- `-j`: 支持bzip2解压文件
- `-v`: 显示操作过程
- `-l`: 文件系统边界设置
- `-k`: 保留原有文件不覆盖
- `-m`: 保留文件不被覆盖
- `-w`: 确认压缩文件的正确性
- `-p`/`--same-permissions`: 用原来的文件权限还原文件
- `-p`/`--absolute-names`: 文件名使用绝对名称, 不移除文件名称钱的"/"号
- `-N`<日期格式> / `--newer`=<日期时间>: 只将较指定日期更新的文件保存到备份文件里
- `--exclude`=<范本样式>: 排除符合范本样式的文件

# 参数

- 文件或目录: 指定要打包的文件或目录列表

# 实例

## 将文件全部打包成`tar`包

- 仅打包, 不压缩

```
    tar -cvf log.tar log.log
```

- 打包后, 以 `gzip` 压缩

```
    tar -zcvf log.tar.gz log.log
```

- 打包后, 以 `bzip2` 压缩

```
    tar -jcvf log.tar.ba2 log.log
```


## 查阅上诉`tar`包内有哪些文件

```
    tar -ztcf log.tar.gz
```

## 将`tar`包解压缩

```
    tar -zxvf /test/log.tar.gz
```

## 只将`tar`内的部分文件解压出来

```
    tar -zxvf /test/log.tar.gz log.log
```

## 文件备份下来, 并且保存其权限

```
    tar -zcvpf log.tar.gz  log2017.log log2018.log
```

## 在文件夹中, 比某个日期新的文件才备份

```
    tar -N "2018/06/17" -zcvf log18.tar.gz test
```

## 备份文件夹内容是排除部分文件

```
    tar --exclude scf/service -zcvf scf.tar.fz scf/*
```


# 最简单的使用 tar

- 压缩

```
    tar -jcv -f filename/tar/bz2 要被压缩的文件或目录名称
```

- 解压缩

```
    tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录
```

- 查询

```
    tar -jtv -f filename.tar.bz2
```

