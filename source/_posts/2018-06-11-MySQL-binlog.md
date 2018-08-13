---
title: MySQL bin日志
date: 2018-06-11 16:09:53
categories: MySQL
tags: [MySQL, binlog]
---

> **mysqlbinlog**是从二进制日志读取语句的工具。
在二进制日志文件中包含的执行过的语句的日志可用来帮助从崩溃中恢复

<!-- more -->

# 查看配置

- 查看`binlog`是否开启

```
    show variables like 'log_bin';
```

- 查看`binlog`文件的目录位置

```
    show variables like 'datadir';
```

# 开启`Binlog`

- `Windows`下

```
    // my.ini
    [mysqld]
    log-bin=mysqlbin-log  # =号后面的名字自定义
```

- `Linux`下

```
    // my.conf
    [mysqld]
    log-bin=/var/log/mysql/mysql-bin.log
```

# 基础使用

```
    mysqlbinlog --start-datetime="2018-06-09 17:50:00" --stop-datetime="2018-06-09 18:00:00" mysql-bin.000009
```

> bin.000009 是当前的binlog文件(使用 show master status可查询)

## 查看 `binlog`

```
    mysqlbinlog -vv --base64-output=decode-rows /path/to/mysql-bin-log
```

## 恢复成 `SQL`

- 恢复本地 `binlog`

```
    mysqlbinlog \
    --no-defaults \                                 #不要阅读任何选项文件
    --base64-output=AUTO \                          #base64输出=自动
    --verbose \                                     #重建为SQL语句
    -- set-charset=utf8 \                           #将字符集添加到输出
    --start-datetime='2018-06-11 00:00:00' \        #开始时间
    --stop-datetime='2018-06-12 00:00:00' \         #结束时间
    ./mysql-bin.000009 > /data/restore.sql          #存储
```

- 恢复远程服务器 `binlog`

```
    mysqlbinlog \
    --no-defaults \                                 #不要阅读任何选项文件
    -uroot -p \                                     #用户名/密码
    -hrm-wz93132ahz16ttyhq.mysql.rds.aliyuncs.com \ #服务器地址
    --read-from-remote-server \                     #从远程服务器读取,
    mysql-bin.000009 > restore.sql                  #存储
```

# 小插曲

不到迫不得已不用binlog日志,当你要使用binlog日志的时候说明你的数据已经丢失了.

可以把错误扼杀在摇篮.

使用`mysqldump`进行备份.

## 导出所有数据库

```
    mysqldump -uroot -proot --all-databases > /all.sql
```

## 导出部分数据库全部数据

```
    mysqldump -uroot -proot --databases db1 db2 > /db.sql
```

## 导出数据库部分表数据

```
    mysqldump -uroot -proot db1 --tables a1 a2 > /table.sql
```

## 条件导出

```
    mysqldump -uroot -proot -databases db1 --tables a1 --where='id=1' > where.sql
```

## 导出数据后生产新的`binlog`文件

```
    mysqldump -uroot -proot --databases db1 -F > binlog.sql
```

## 只导出表结构不导出数据

```
    mysqldump -uroot -proot --no-data --databases db1 > nodata.sql
```

## 跨服务器导出导入数据

```
    msyqldump --host=h1 -uroot -proot --databases db1 | mysql --hosts=h2 -uroot -proot db2
```

**将h1服务器中的db1数据库的所有数据导入到h2中的db2数据库中，db2的数据库必须存在否则会报错**

```
    mysqldump --host=h1 -uroot -proot -C --databases db1 |mysql --host=h2 -uroot -proot db2 
```

> 加上-C参数可以启用压缩传递。

## 锁表导出 `--lock-tables`

开始导出前，锁定所有表。用`READ LOCAL`锁定表以允许`MyISAM`表并行插入。
对于支持事务的表例如`InnoDB`和`BDB`，`--single-transaction`是一个更好的选择，因为它根本不需要锁定表。

请注意当导出多个数据库时，`--lock-tables`分别为每个数据库锁定表。
因此，该选项不能保证导出文件中的表在数据库之间的逻辑一致性。不同数据库表的导出状态可以完全不同。

## 导出存储过程和自定义函数`--routines, -R`

```
    mysqldump -uroot -proot -host=localhost --all-databases --routines
```

## 压缩备份

```
    // 压缩
    mysqldump -uroot -proot -databases abd 2>/dev/null | gzip >/adc.sql.gz
    // 还原
    gunzip -c abc.sql.gz | mysql -uroot -proot abc
```

# 参考

- [mysqlbinlog - 处理二进制日志文件的实用程序](https://dev.mysql.com/doc/refman/5.7/en/mysqlbinlog.html)
- [MySQL mysqldump数据导出详解](http://www.cnblogs.com/chenmh/p/5300370.html)