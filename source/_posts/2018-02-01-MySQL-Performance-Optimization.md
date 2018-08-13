---
title: MySQL「性能优化」
date: 2018-02-01 15:17:44
categories: MySQL
tags: [MySQL, 优化]
---

数据库优化与检查是一项比较艰难的任务，必须借助于一些工具，而且往往问题比较隐蔽，多是在部署在生产环境一段时间之后，因为数据库的性能瓶颈，导致一系列问题的产生。

<!--more-->

# 常用参数

- 查看mysql是否开启慢查询日志

```
    show variables like 'slow_query_log';
    set global slow_query_log=on; //开启慢查日志
```

- 设置没有索引的记录到慢查询日志

```
    set global log_queries_not_using_indexes=on;
```

- 查看超过多长时间的sql进行记录到慢查询日志
    
```
    show variables like 'long_query_time';
    set global long_query_time=1;
```


# 慢查日志分析工具

## mysqldumpslow

使用`mysqldumpslow -h`来查看帮助信息
使用`mysqldumpslow -t 3 /home/mysql/data/mysql-slow.log | more`来对日志进行分析。

## pt-query-digest

输出到文件：

```
    pt-query-digest show.log > slow_log.repor
```

# 如何通过慢查日志发现有问题的SQL？

##  查询次数多且每次查询占用时间长的SQL

通常为`pt-query-digest`分析的前几个查询

##  IO大的SQL，数据库的主要瓶颈就在于IO

注意pt-query-digest分析中的 `Rows examine` 项。**扫描行数多，占用io大**

## 未命中索引的sql

注意`pt-query-digest`分析中**Rows examine（扫描行数）**和**Rows send（发送行数）**的对比
**如果这个比值比较大，说明索引命中率不高**


# 怎么优化有问题的sql

## 使用explain查询SQL的执行计划

```
    explain select * from user;
```

- `table`：表名；
- `type`：连接的类型，`const`、`eq_reg`、`ref`、`range`、`index`和`ALL`；
  - const：主键、索引;
  - eq_reg：主键、索引的范围查找;
  - ref：连接的查找（join）;
  - range：索引的范围查找;
  - index：索引的扫描;
  - All：表扫描
- `possible_keys`：可能用到的索引；
- `key`：实际使用的索引；
- `key_len`：索引的长度，越短越好；因为mysql每次的读取都是以页为单位，一页中的索引越大，查询的效率就越高
- `ref`：索引的哪一列被使用了，常数较好；
- `rows`：mysql认为必须检查的用来返回请求数据的行数；
- `extra`：using filesort、using temporary（常出现在使用order by时）时需要优化，因为都是用了外部文件或者临时表的存储

## 选择合适的索引列

- 在`where`，`group by`，`order by`，`on`从句中出现的列

- 索引字段越小越好(因为数据库的存储单位是页，一页中能存下的数据越多越好 )
  索引字段越小越好，是因为数据库的存储是以页为单位，一页数据存储越大，获取的数据也越多，对我们io的效率也更好
  
- 离散度大得列放在联合索引前面

计算离散度:

```
    select count(distinct customer_id), count(distinct staff_id) from payment;
```

查找字段的唯一值（不同数的个数）来确定count（distinct customer_id）离散度的大小来确定索引建立在哪一个列上.

离散度，我的理解就是唯一性了，比如主键，绝对是离散度最大的，而一些用来标识状态标识的列，基本只有几个可选项，离散度就很小

**index(col1,col2)** 在建立联合索引时，离散度高的列放到前面


## 冗余索引检查

索引存在的目的是为了加快查询的效率，不过不是索引越多越好，建立索引要适当才好。

过多的索引会增加数据库判断使用什么索引来查询的开销，所以，有时候也会出现以去掉重复或者无效的索引为优化手段的优化方式。

冗余索引是指多个索引的前缀列相同，或是在联合索引中包含了主键的索引，下面这个例子中key(name, id)就是一个冗余索引

```
    create table test(id int not null primay key,name varchar(10) not null,title varchar(50) not null, key(name,id))engine=innodb;
```

主键索引本来就是唯一索引

### 索引检查语句

```
    use information_schema;
    SELECT a.TABLE_SCHEMA AS 'db'
        ,a.TABLE_NAME AS 'table'
        ,a.INDEX_NAME AS 'index1'
        ,b.INDEX_NAME AS 'index2'
        ,a.COLUMN_NAME AS 're_col'
    FROM STATISTICS a
        JOIN STATISTICS b ON a.TABLE_SCHEMA = b.TABLE_SCHEMA
    AND a.TABLE_NAME = b.TABLE_NAME
        AND a.SEQ_IN_INDEX = b.SEQ_IN_INDEX
        AND a.COLUMN_NAME = b.COLUMN_NAME
    WHERE a.SEQ_IN_INDEX = 1 AND a.INDEX_NAME <> b.INDEX_NAME
```

### 索引检查工具 `pt-duplicate-key-checker`

查询冗余索引工具 `pt-duplicate-key-checker -uroot -p '123456' -h 127.0.0.1`

## 字段类型优化

- 可以使用int类型存储时间类型：
  - int类型转时间类型函数：`from_unixtime()`
  - 时间类型转int类型：`unix_timestemp()`

- 可以使用bigint存储ip地址
  - bigint类型转ip函数：`inet_aton()`
  - ip转bigint类型：`inet_ntoa()`
  

# 参考

- [Mysql性能优化笔记](http://jinfang.life/posts/a8073527/)