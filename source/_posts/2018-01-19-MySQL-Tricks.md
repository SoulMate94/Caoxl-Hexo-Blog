---
title: MySQL  Tricks
date: 2018-01-19 17:41:44
categories: MySQL
tags: [MySQL, Database]
---

持续记录使用 MySQL 过程中遇到过的偏冷知识点和坑。

# CLI

## enable `mysql` command on mac

```
    # 1. install mysql workbench
    brew cask install mysqlworkbench
    
    # 2. export environment variables
    export PATH=$PATH:/Applications/MySQLWorkbench.app/Contents/MacOS
    
    # 3. use
    mysql -h {host} -u {user} -p {password}
```

<!--more-->

## mycli
    
```
    pip install mycli
    
    brew install mycli
```

有语法提示，简直不能太赞。使用和普通 mysql 命令一样。

## over SSH Tunnel

```
    # -f => background
    # -L => bind port between local and remote
    # -N => don't execute command just forwarding ports
    
    ssh -f user@ssh.example.com -L 3307:mysql_server.example.com:3306 -N
    mysql -h 127.0.0.1 -P 3307 -u<mysql_user> -p <mysql_user_password>
    
    # Kill SSH Tunnel progress
    # Get pid
    
    netstat -vanp tcp | grep 3307
    lsoif -i:3307
    kill -9 {pid}
```

# 语法

## NULL vs NOT NULL

除非真的要保存 NULL，否则尽量避免使用 NULL。

但把 NULL 列改为 NOT NULL 带来的性能提升很小，所以除非确定它引入了问题，否则就不要把它当作优先的优化措施。

## index with nullable field

当 where 条件是 `column = cond` ，column 是 `is null`，MySQL 也能对其进行优化，即 MySQL 可以使用索引来查找 `nullable` 字段。

但是，当 where 条件种包含如下情况时，该优化不会生效：

```
    SELECT * FROM user WHERE `name` IS NULL
```

其中，`name` 字段的定义是 NOT NULL。该情况通常出现在联合查询时。

> <u>https://dev.mysql.com/doc/refman/5.7/en/is-null-optimization.html</u>

## FROM DUAL

```
    SELECT DATABASE() FROM DUAL;
```

> There is much discussion over whether or not FROM DUAL should be included in this or not. On a technical level, it is a holdover from Oracle and can safely be removed. If you are inclined, you can use the following instead: `SELECT DATABASE()`;
>  
> That said, it is perhaps important to note, that while FROM DUAL does not actually do anything, it is valid MySQL syntax. From a strict perspective, including braces in a single line conditional in JavaScript also does not do anything, but it is still a valid practice

## `having` vs `where`

```
    create table `user` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `name` varchar(16) NOT NULL,
      `gender` tinyint NOT NULL,
      `age` tinyint NOT NULL,
      PRIMARY KEY (`id`)
    );
```

`having` 使用的是从已经筛选出来过的字段（包括别名），而 `where` 使用的是表中定义好的字段。

```
    select name from user where age > 18;    -- 不会报错
    
    select name from user having age > 18;   -- 会报错 `Unknown column 'age' in 'having clause'`
    
    select * from user having age > 18;    -- 不会报错, `*` 已包含了 `age`
    
    -- 不会报错
    select name, avg(age) as avg_age from user group by gender having avg_age > 18;
    
    -- 报错，表 user 不存在 `avg_age` 字段
    select name, avg(age) as avg_age from user where avg_age > 18 group by gender;
```

`where` 性能高于 `having`，能写在 `where` 限定条件中的就尽量写在 `where` 中。

## `group by` 的方便和坑

- 方便：统计不同性别的用户个数（同一列中不同值的个数）：

```
    -- 1. count
    select
    	count(gender = 1 OR NULL) as boy,
    	count(gender = 2 OR NULL) as girl,
    	count(gender = 3 OR NULL) as gay,
    from user
    
    -- 2. sum
    select
    	sum(if(gender = 1, 1, 0)) as boy,
    	sum(if(gender = 2, 1, 0)) as girl,
    	sum(if(gender = 3, 1, 0)) as gay
    from user;
    select
    	sum(gender = 1) as boy,
    	sum(gender = 2) as girl,
    	sum(gender = 3) as gay
    from user;
    
    -- 4. group by
    select gender, count(*) as cnt from user group by gender;
```

- 坑：性能

如果没有用到索引数据量一般大也会巨慢，甚至出现 MySQL CPU 爆高，超时等。

`group by`与 `order by` 的索引优化基本一样，`group by` **实质是先排序后分组**，也就是分组之前必排序，遵照索引的最佳左前缀原则可以大大提高 group by 的效率。

# 隐式特性（坑）

## Implict Commit

### 部分事务隐式强制性提交

> [Some databases, including MySQL, automatically issue an implicit COMMIT when a database definition language (DDL) statement such as DROP TABLE or CREATE TABLE is issued within a transaction. The implicit COMMIT will prevent you from rolling back any other changes within the transaction boundary.](http://www.php.net/manual/en/pdo.begintransaction.php)
> 
> CREATE TABLE操作背后涉及到了大量的操作，不仅仅包括对核心表的操作，还包括大量内存数据结构的更新（如Schema），以及存储系统的变更（如创建相应的数据块），工程上很难把这些操作做成原子的。
> 
> 那么，应该如何做呢？比较折中的方式就是跟用户做一个约定：CREATE TABLE操作总默认COMMIT它之前的事务，这就是implict commit。

详情可参考 <u>[这篇博客](http://blog.csdn.net/maray/article/details/46410817)</u>。


# binlog

## 查看 binlog

```
    mysqlbinlog -vv --base64-output=decode-rows /path/to/mysql-bin-log
```

## 恢复成SQL

```
    # 恢复本地 binlog
    mysqlbinlog \
    --no-defaults \
    --base64-output=AUTO \
    --verbose \
    --set-charset=utf8 \
    --start-datetime='2017-07-16 00:00:00' \
    --stop-datetime='2017-07-17 00:00:00' \
    ./mysql-bin.000611 > /data/restore.sql
    # 恢复远程服务器 binlog
    mysqlbinlog \
    --no-defaults \
    -uwaimai_o2o -p \
    -hrm-wz93132ahz16ttyhq.mysql.rds.aliyuncs.com \
    --read-from-remote-server \
    mysql-bin.000618 > restore.sql
```

> https://dev.mysql.com/doc/refman/5.7/en/mysqlbinlog.html

## 使用不小于 MySQL server 的版本

如果 mysqlbinlog 版本太低，则会出现类似报错如下：

```
    ERROR: Error in Log_event::read_log_event(): 'Sanity check failed', data_len: 151, event_type: 35
    ERROR: Could not read entry at offset 120: Error in log format or read error.
```

# 版本有关问题

- **Specified key was too long; max key length is 767 bytes**

> 767 bytes is the stated prefix limitation for InnoDB tables in MySQL version 5.6 (and prior versions). It’s 1,000 bytes long for MyISAM tables. In MySQL version 5.7 and upwards this limit has been increased to 3072 bytes.
>  
> <u>https://stackoverflow.com/questions/1814532/1071-specified-key-was-too-long-max-key-length-is-767-bytes</u>

## 备份/恢复

```
    mysqldump -hrm-wz93132ahz16ttyhq.mysql.rds.aliyuncs.com -uwaimai_o2o -p waimai_020 | gzip > wz93132ahz16ttyhq-2017-11-23.sql.gz
```

# 主机连接

## 127.0.0.1 vs localhost

使用 mysql cli 客户端连接 docker 容器中的 mysql，使用 `127.0.0.1` 可以连接上，但是使用 `localhost` 却不能连：

```
    mysql -h 127.0.0.1 -uroot -p*****    # Success
    mysql -h localhost -uroot -p*****    # Failed
    ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
```

其实这个问题不属于 MySQL 范围内的问题，所有服务器程序（ssh, telnet…）都会遇到这些问题。

根本原因是由服务端程序实际上监听的地址是一个 IPv4 还是一个 hostname。

比如这里的 MySQL，如果 MySQL server 只是监听了 127.0.0.1 这个 IPv4 地址，那么客户端就只能使用 127.0.0.1 这个 IP 连接。

此外，从错误信息中还可以总结一点，unix-like 操作系统默认会把 localhost 当作 UNIX domain socket，而 windows 默认就认为是 TCP/IP，这样的结果是：

- 在 unix-like OS 上，如果指定地址为 localhost 那么 OS 会默认常识使用 Unix domain socket；如果指定地址为 `127.0.0.1`，那么 OS 才会使用 TCP/IP。
- 在 windows 上，无论指定 `localhost` 还是 `127.0.0.1`，OS 都会使用 TCP/IP。


> UNIX socket is an inter-process communication mechanism that allows bidirectional data exchange between processes running on the same machine.
>  
>  IP sockets (especially TCP/IP sockets) are a mechanism allowing communication between processes over the network. In some cases, you can use TCP/IP sockets to talk with processes running on the same computer (by using the loopback interface).
  
> UNIX domain sockets know that they’re executing on the same system, so they can avoid some checks and operations (like routing); which makes them faster and lighter than IP sockets. So if you plan to communicate with processes on the same host, this is a better option than IP sockets.
>  
> <u>https://serverfault.com/questions/124517/whats-the-difference-between-unix-socket-and-tcp-ip-socket</u>

# 参考
 - [MySQL :: MySQL 5.7 Reference Manual :: B.5.4.3 Problems with NULL Values](https://dev.mysql.com/doc/refman/5.7/en/problems-with-null.html)
 - [Are many NULL columns harmful in mysql InnoDB?](https://dba.stackexchange.com/questions/15335/are-many-null-columns-harmful-in-mysql-innodb)
 - [how to know the date of insertion of record in SQL](https://stackoverflow.com/questions/30367982/how-to-know-the-date-of-insertion-of-record-in-sql)
 - [SQL How to replace values of select return?](https://stackoverflow.com/questions/16753122/sql-how-to-replace-values-of-select-return)
 - [MySQL connection over SSH tunnel - how to specify other MySQL server?](https://stackoverflow.com/questions/18373366/mysql-connection-over-ssh-tunnel-how-to-specify-other-mysql-server)
 - [PDO: Transactions don’t roll back?](https://stackoverflow.com/questions/2426335/pdo-transactions-dont-roll-back)
 - [Difference between SET autocommit=1 and START TRANSACTION in mysqls](https://stackoverflow.com/questions/2950676/difference-between-set-autocommit-1-and-start-transaction-in-mysql-have-i-misse)
 
 