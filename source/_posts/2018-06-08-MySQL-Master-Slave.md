---
title: MySQL 主从复制
date: 2018-06-08 14:05:59
categories: MySQL
tags: [MySQL]
---

>MySQL数据库自身提供的主从复制功能可以方便的实现数据的多处自动备份，实现数据库的拓展。
多个数据备份不仅可以加强数据的安全性，通过实现读写分离还能进一步提升数据库的负载性能。

<!-- more -->

# 主从复制/读写分离模型

![Master-Slave](http://www.caoxl.com/images/MasterSlave.jpg)

Mysql主从复制的实现原理图大致如下(来源网络)：

![主从复制](http://www.caoxl.com/images/主从复制.jpg)

MySQL之间数据复制的基础是二进制日志文件（binary log file）

# 实现原理
一台MySQL数据库一旦启用二进制日志后，其作为`master`，它的数据库中所有操作都会以“事件”的方式记录在二进制日志中，
其他数据库作为`slave`通过一个I/O线程与主服务器保持通信，并监控`master`的二进制日志文件的变化，
如果发现`master`二进制日志文件发生变化，则会把变化复制到自己的中继日志中，然后`slave`的一个SQL线程会把相关的“事件”执行到自己的数据库中，以此实现从数据库和主数据库的一致性，也就实现了主从复制。


# 实现配置

## 主服务器

- 开启二进制日志
- 配置唯一的`server-id`
- 获得`master`二进制日志文件名及位置
- 创建一个用户`slave`和`master`通信的用户账号

```
    [mysqld]
    server-id   = 1
    log_bin     = /var/log/mysql/mysql-bin.log
```

## 从服务器

- 配置唯一的`server-id`
- 使用master分配的用户账号读取master二进制日志
- 启用`slave`服务

```
    [mysqld]
    server-id = 2
```

# 具体操作

## 准备工作

- 主从数据库版本最好一致
- 主从数据库内数据保持一致
- 主数据库: `47.91.221.85`/`linux`
- 从数据库: `106.15.38.11`/`linux`

## 主服务器`master`修改

- 修改`mysql`配置

找到主数据库的配置文件my.cnf(或者my.ini),插入以下数据

```
    [mysqld]
    log-bin = /var/log/mysql/mysql-bin.log #开启二进制日志
    server-id = 99 #设置server-id 唯一即可
```

- 重启`mysql`, 创建用于同步的用户账号

```
    service mysql restart
    
    // mysql -u root -p 进入mysql
    // 创建用户,IP为可访问该master的IP，任意IP就写'%'
    mysql> CREATE user 'cxl@106.15.38.11' IDENTIFIED By '110119';
    
    // 分配权限,IP为可访问该 master的IP，任意IP就写'%'
    mysql> GRANT REPLICATION SLAVE On *.* TO 'cxl@106.15.38.11';
    
    // 刷新权限
    mysql> flush privileges;
```

- 查看`master`状态,记录二进制文件名(mysql-bin.000009)和位置(421)

```
    mysql> show master status;
    +------------------+----------+--------------+------------------+
    | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
    +------------------+----------+--------------+------------------+
    | mysql-bin.000009 |      421 |              |                  |
    +------------------+----------+--------------+------------------+
```


## 从服务器`Slave`修改

- 修改`mysql`配置

同样找到`my.cnf`配置文件，添加`server-id`

```
    [mysqld]
    server-id = 66 #设置server-id，必须唯一
```

- 重启`mysql`,打开`mysql`,执行同步`SQL`语句

(需要主服务器主机名，登陆凭据，二进制文件的名称和位置)：

```
    mysql> CHANGE MASTER TO
        ->     MASTER_HOST='47.91.221.85',  // 主服务器地址(外网地址)
        ->     MASTER_USER='cxl',           // 用户
        ->     MASTER_PASSWORD='110119',    // 密码
        ->     MASTER_LOG_FILE='mysql-bin.000009',
        ->     MASTER_LOG_POS=421;
```

- 启动`slave`同步进程

```
    mysql>start slave;
```

- 查看`slave`状态

```
    mysql> show slave status\G;
    *************************** 1. row ***************************
                   Slave_IO_State: Waiting for master to send event
                      Master_Host: 47.91.221.85
                      Master_User: cxl
                      Master_Port: 3306
                    Connect_Retry: 60
                  Master_Log_File: mysql-bin.000009
              Read_Master_Log_Pos: 11662
                   Relay_Log_File: mysqld-relay-bin.000022
                    Relay_Log_Pos: 11765
            Relay_Master_Log_File: mysql-bin.000013
                 Slave_IO_Running: Yes
                Slave_SQL_Running: Yes
                  Replicate_Do_DB: 
              Replicate_Ignore_DB: 
            ...
```

当`Slave_IO_Running`和`Slave_SQL_Running`都为YES的时候就表示主从同步设置成功了。

`master`开启二进制日志后默认记录所有库所有表的操作，可以通过配置来指定只记录指定的数据库甚至指定的表的操作，
具体在mysql配置文件的`[mysqld]`可添加修改如下选项：

```
    # 不同步哪些数据库  
    binlog-ignore-db = mysql  
    binlog-ignore-db = test  
    binlog-ignore-db = information_schema  
      
    # 只同步哪些数据库，除此之外，其他不同步  
    binlog-do-db = game 
```

查看`master`状态时就可以看到只记录了`test`库，忽略了`manual`和`mysql`库。

# 参考

- [CentOS下MySQL主从复制(Master-Slave)实践](https://blog.csdn.net/yifanSJ/article/details/79281016)
