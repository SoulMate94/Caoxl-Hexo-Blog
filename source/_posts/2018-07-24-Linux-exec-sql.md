---
title: Linux服务器执行sql文件
date: 2018-07-24 14:28:32
categories: Linux
tags: Linux
---

> Linux服务器执行sql文件

<!-- more -->

# 连接数据库

```
    mysql -u user -p password
```

# 新建数据库

```
    create database database_name;
    show databases;
```

# 选择数据库

```
    source /test.sql
    show tables;
```

# 删除数据库

```
    drop database dabatase_name;
```


> 备份数据库查看 [MySQL bin日志](http://blog.caoxl.com/2018/06/11/MySQL-binlog/)
