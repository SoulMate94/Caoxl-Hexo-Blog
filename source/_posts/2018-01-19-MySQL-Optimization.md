---
title: MySQL  优化
date: 2018-01-19 12:58:48
categories: MySQL
tags: [MySQL, 优化]
---

> 主要从概念上总结下 MySQL 优化的思路。
 
# 数据库优化简介

## Why ?

### 避免出现页面访问错误

- 数据库连接 timeout 发生 50x 错误
- 由于慢查询造成页面无法加载
- 由于阻塞造成数据无法提交

<!--more-->  

### 增加数据库的稳定性

很多数据库问题都是由低效的查询引起的。

### 优化用户体验

- 页面的流畅访问
- 网站功能的良好体验


## How ?

### 硬件

CPU 核数、更多的内存和更快的 I/O(固态硬盘等)，但是 CPU 并不是越多越好，因为 MySQL 的有些查询只能是单核完成。

此外，I/O 越快也并不能保证减少数据库锁的问题，因为锁是 MySQL 等多数数据库内部为保证数据完整性的一种机制，因此无法完全避免掉阻塞。

可见，硬件的优化是成本最高但是效果最差的，如果底层的优化不够，产生了 太多的慢查询和阻塞，那么就会带来高并发， 高并发的场景下，再好的硬件也会有很高的 loading。

### 系统配置

MySQL 的运行环境大部分都是 Linux 系统，所以会受到 Linux 系统本身的一些限制。

此外， TCP/IP 连接数、打开的文件数(I/O操作频率)，也是会对 MySQL 的性能造成限制。

### 数据库表结构

良好的表结构是写出良好的 SQL 语句的基础，我们要根据数据库设计范式来设计表结构，尽量考虑如何设计表结构才有利于查询，尽量简洁明了，尽量减少数据的冗余。

### SQL 及索引

这是最常见的优化方式，我们需要根据需求写出结构良好的 SQL，然后写出结构良好的 SQL 语句。因为同一个业务可能可以用多种 SQL 来完成，但是尽量选择结构最好的 SQL。

> 以上的 4 个思路，从下往上，成本递增，但是效果却在递减。

# 服务器硬件优化

> 提示：由于成本和效率不太成正比，因此不是首选优化点。

## 如何选择 CPU

> 是选择更快的单核，还是选择核数更多的 CPU

可以参考的依据有：

- 对 MySQL 来说，有些工作只能用到单核 CPU。比如：Replicate, SQL 等。

- MySQL 对 CPU 核心数的支持并不是越多越快，详见官方手册。

> MySQL 5.5 使用的服务器不要超过 32 核。国外有人(Facebook工程师)测试过，MySQL 5.5 Community Version，CPU 超过 32 核心后性能不升反降。

- 频率高的单核同样能提高性能

## Disk I/O 优化

### 常用 RAID 级别

#### RAID0

也称“条带”，就是把多个磁盘连接成一个硬盘使用，这个级别的 IO 最好。

#### RAID1

也称“镜像”，要求至少有两个磁盘，每组磁盘存储的数据相同。

#### RAID5

即把多个（最少 3 个）的硬盘，合并成 1 个逻辑盘使用，数据读写时会建立奇偶校验信息，并且奇偶校验信息和相对应的数据分别存储在不同的磁盘上。

当 RAID5 的一个磁盘数据发生损坏后，利用剩下的数据和相应的奇偶校验信息去恢复被破坏的数据。

#### RAID1+0

即 RAID1 和 RAID0 的结合，同时具备两个级别的优缺点。

由于 RAID0 安全性不好，RAID1 I/O 不好，因此，一般数据库都是使用这个级别。

### SNA 和 NAT

> SNA 和 NAT 是否适合数据库？

SNA 和 NAT 的优点：

- 常用于高可用解决方案
- 顺序读写效率高（但是随机读写差）

这两种适合做磁盘矩阵，容灾性很好，但是从提升数据库 I/O 性能的角度来看，并不适合数据库读写的特点，因为数据库随机读写的比率很高。

因此，对于网络存储设备，需要测试后才决定是否采用。

# 系统配置优化

## 操作系统配置优化

数据库是基于 OS 的，所以 OS 的一些参数配置是影响 MySQL 性能的基础。

目前 MySQL 主要都安装在 GUN/Linux 上，因此下面说的都是在说 Linux，其常用的系统配置如下：

### 网络方面

修改 `/etc/sysctl.conf`：

```
    # 增加 TCP 支持的队列数
    net.ipv4.tcp_max_syn_backlog = 65535
    
    # 减少断开连接时的资源回收
    net.ipv4.tcp_max_tw_buckets = 8000
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_fin_timeout = 10
```

### 打开文件数限制

```
    # 查看目录的各位限制
    ulimit -a
    
    # 修改 /etc/security/limits.conf 修改系统对打开文件数的限制
    * soft nofile 65535
    * hard nofile 65535
```

### 防火墙等安全工具

```
    在 MySQL 服务器上最好关闭 iptables/selinux 等防火墙软件，因为对 MySQL 服务器来说，最好使用硬件防火墙来屏蔽而不是使用软件防火墙。
```

### 文件系统类型

选择文件系统类型参考以下因素：

- I/O 操作性能
- 并发支持
- 多文件下工作
- 文件切片
- 崩溃恢复时间（最好选择日志文件系统）
- 安全
- 影响性能的可禁用选项

## MySQL 配置

### 配置文件

有两种方式可以配置 MySQL 服务器：

- 启动时指定配置参数
- 使用配置文件启动

大多数情况下，MySQL 的配置文件位于 `/etc/my.cnf`、`/etc/mysql/my.cnf`、`C:/windows/my.ini`，可以手动查找配置文件顺序：

```
    /usr/sbin/mysqld --verbose --help | grep -A 'Default options'
```

配置文件顺序以最后的为准，即如果存在多个配置文件时，后面配置的会覆盖前面的配置。

### Innodb 常用配置选项参数

- **innodb_buffer_pool_size**

用于配置 Innodb 缓冲池。

如果数据库中只有 Innodb 表，则推荐配置量为总内存的 75%。

**举例说明：**

```
    select engine,
    round(SUM(data_length + index_length)/1024/1024, 1) as 'Total MB',
    from information_schema.TABLES
    where table_schema not in ('information_schema', 'performance_schema')
    group by engine;
```

innodb_buffer_pool_size 通常需要 >= 这个查询结果中的 `Total MB`。

`Total MB` 代表的是所有 Innodb 表的大小+索引的大小之和，当系统内存有限时，该值需要尽可能的大。

- **innodb_buffer_pool_instances**

这是 MySQL 5.5 中新增的参数，可以控制缓冲池的个数。（默认情况下只有一个）

在 MySQL 中有些资源是被独占的，比如缓冲池。增加缓冲池的个数有利于提高并发性能，降低阻塞的频率。

> **每个缓冲池实例的大小 = innodb_buffer_pool_size / innodb_buffer_pool_instances。**

- **innodb_log_buffer_size**

是 innodb log 缓冲的大小。由于日志最长每秒钟就会刷新，所以一般不用大大。

> 为了效率，Innodb 的日志是先写入缓冲区，然后再写入磁盘文件的 不用设置太大，只要能够存放下一秒内事物的日志就行了。

- **innodb_flush_log_at_trx_commit**

该参数决定的是数据库多久把变更刷新到磁盘：

- 为 0：表示每次提交不进行刷新，每一秒才把变更刷新一次
- 为 1：表示每次提交都会把变更刷新到磁盘( 最安全 )
- 为 2：表示每次提交只是把变更刷新到缓冲区，然后每一秒从缓冲区刷新到磁盘，最多丢失 1 秒的提交(为了 I/O 性能一般都是设置为 2)

关键参数之一，对于 Innodb 的 I/O 效率影响很大。

默认为 1，可以取 0、1、2 三个值，一般为 2，如果对数据安全性要求高则使用默认值 1。

- **innodb_read_io_threads** 和 **innodb_write_io_threads**

这两个参数决定了 Innodb 表的读写 I/O 进程数，默认为 4。

该参数在 5.5 后可以人为地进行调整，当然也要考虑读写负载的实际情况考虑，以提高 Innodb 并发读写的线程数。

- **innodb_file_per_table**

控制 Innodb 表的每个表使用的独立表空间，默认为 `OFF`，表示默认所有表都会建立在共享表空间中。关键参数之一。

虽然默认是没启用的，但是建议设置为 On。如果所有 Innodb 表都使用共享表空间，那么也会带来 I/O 瓶颈，因为是顺序读写共享表空间的其次，Innodb 的共享表空间是无法单独收缩的，所以如果删除了一个很大的表，这时 候如果想收缩该共享表空间的话，只能先导出所有数据再重新导入进去。

如果每张表使用的磁盘空间都是独立的，那么当对某张表进行了删除等操作，MySQL 就会马上回收这部分磁盘空间，另外，由于对多个表文件的读写分开了，也提高了读写 的并发性。


- **innodb_stats_on_metadata**

决定了 MySQL 在什么情况下会刷新 Innodb 表的统计信息。

数据库优化器在使用表的索引的时候，会借助表的统计信息，定时刷新表的统计信息就能够确保优化器的正常读取，但是太高也会影响 MySQL 的性能。

默认情况，MySQL 在查询表结构的时候( `show create table / show table status` ) 都会刷新表的统计信息，而这是不必的，所以这种情况下是不必启用的，通常情况对该统计信息的刷新都是在数据库闲时进行手动刷新的，这样也可以保持数据库的性能。

为维护配置文件效率比较低，也可以借助第三方配置工具来完成的。

### 第三方配置工具

- <u>[Percon Configuration Wizard](https://tools.percona.com/sign-in)</u>

这里可以手动选择配置选项，然后根据你的选择为你生成一份配置文件，并发送到你的邮箱。


# 数据库结构优化

## 如何选择合适的数据类型

- 使用可以存下数据的最小的数据类型（增加单页数据量，减少 I/O 次数）
- 使用简单的数据类型（mysql 处理 int 要比 char/varchar 简单）
- 尽可能使用 not null 定义字段（Innodb 表字段为 NULL 时会占用额外的高位，从而减少实际存储空间）
- 尽量少用 text 类型，非用不可时最好考虑分表（text 和 blob 类型往往专门存储在附加表中）

## 应用举例

### 使用 int 存储日期和时间

同时，使用 from_unixtime() 和 unix_timestamp() 来转换，两者往往配合使用。

```
    select from_unixtime(unix_timestamp());
```

### 使用 bitint 来存 IP 地址

同时，使用 inet_aton() 和 inet_ntoa() 来转换，两者往往互逆使用。

```
    select inet_aton('192.168.0.1');    -- 3232235521
    select inet_ntoa(3232235521);       -- 192.168.0.1
```

一个 IPv4 地址最多占用 15 个字节，而 bigint 占用 8 个字节，还是很划算的。

使用 bigint 比 char/varchar 类型更高效，因为数字比字符计算速度快。

## 表的范式化和反范式化

### 范式化

范式化是指设计数据库时候需要遵从的规范，目前说到数据库范式大部分情况指的都是`第三范式（3NF）`：

- 首先必须满足 `1NF` 和 `2NF`
- 要求表中不存在非关键字段对任何候选关键字段的传递函数依赖，即，不是主键的字段之间不能互相依赖。

### 违背范式化设计带来的问题

- 数据冗余
- 插入异常
- 更新异常
- 删除异常

### 反范式化

为了某些应用场景下查询效率的考虑，把原本符合 3NF 的表，以适当增加冗余的方式，减少多次连表查询的次数，达到优化查询效率的目的。`反范式化可以理解为一种以空间换时间的行为`。


## 表的拆分

### 垂直拆分

把原来有很多列的表拆分成多个表，解决的是表的宽度问题。

通常垂直拆分按以下原则进行：

- 把不常用的字段单独存放到一个表中
- 把大字段／二进制字段独立存放到一个表中
- 把经常一起使用的字段放到一个表中

垂直拆分是更应该在数据表设计之初就执行的步骤。

### 水平拆分

把每一个过大数据量的表，拆分成结构完全相同的子表，解决的是单表数据量过大的问题。

**举例说明:**
 
假设 ID 递增的 100W 规模的表 tb 要水平拆分为 3 张表 tb0, tb_1, tb_2，则通常做法是：

- 创建 3 张结构完全一样的字表

```
    create table tb0 like tb;
    create table tb1 like tb;
    create table tb2 like tb;
```

- 对大表主键取余数，然后将余数相同的记录保存到同一张子表

```
    insert into tb0 (select * from tb where mod(id, 3) = 0);
    insert into tb1 (select * from tb where mod(id, 3) = 1);
    insert into tb2 (select * from tb where mod(id, 3) = 2);
```

- 也可以查询后直接创建子表

```
    create table tb0 (select * from tb where mod(id, 3) = 0);
    create table tb1 (select * from tb where mod(id, 3) = 1);
    create table tb2 (select * from tb where mod(id, 3) = 2);
```

- 对主键取余，获得拆分后的子表

```
    mod(1024, 3);    // 1 => 代表 id 为 1024 的记录在拆分后的第 1 张子表 tb1
```

- 针对不同的余数 CRUD 不同的子表

```
    select * from tb1 where id = 1024;
    delete from tb1 where id = 1024;
```

为了区分业务，不至于造成影响，建议前台用拆分后的表，后台使用汇总表。


# SQL 及索引优化

## 索引

### 如何选择合适的列建立索引？

- 在 where、group by、order by、on 从句中出现的列

有些特殊情况下，select 从句中出现的字段也会建立索引。

- 大小越小的字段，最好是数字

同样是出于较少 I/O 次数的考虑：`MySQL` 中存储数据的单位是页，单页中存储的数据越多，查询效率更高。

- 离散度大的列放到联合索引的前面

**举例说明:**

如果要建立含有 2 个字段 a 和 b 的索引，如果 a 的离散度比 b 大，那么建立的索引应该为：index(a, b)，反之同理。

### 覆盖索引

当一个索引中包含了查询中的所有数据列，那么这种索引就是覆盖索引。这种查询场景下不需要回表操作，因此效率很高。

当查询的频率很高，而且表中包含的列比较少的时候，往往采取覆盖索引来获取查询结果。

### 如何判断字段的离散度？

```
    select count( distinct a ) as res_a, count(distinct b) as res_b from tb;
```

**如果 res_a 大于 res_b 则说明字段 a 的离散度更大。**

> **离散度越大，过滤的数据越多。**

### 索引并非“银弹”

通常情况下，`创建索引能提升查询效率，但是却降低了写入效率( insert/update/delete)`。但是，过多的索引，也会影响查询效率，因为索引过多会使分析更慢。所以，有时候有必要根据情况删除重复的、冗余的索引。

#### 重复索引

是指，相同的列以相同的顺序建立的同类型的索引。

比如，id 字在声明了主键索引 `primary key` 后已经是唯一类型，如果在手动增加一个索引 `unique(id)` 的话，和 `primary key` 就属于重复索引。


#### 冗余索引

指的是，多个索引的前缀列相同，或是在联合索引中包含了主键的索引。举例说明：

```
    create table tb (
    	id int not null primary key,
    	name varchar(10) not null,
    	key(name, id)
    ) engine = innodb;
```

这里由于主键默认情况下就是 id 字段上的索引，所以，**没有必要再在主键上建立索引。**

#### 查找重复、冗余索引

##### 通过 SQL

```
    use information_schema;
    select a.TABLE_SCHEMA as '数据名'
    , a.table_name as '表名'
    , a.index_name as '索引1'
    , b.index_name as '索引2'
    , a.column_name as '重复列名'
    from statistics a
    join statistics b
    on a.table_schema = b.table_schema
    and a.table_name = b.table_name
    and a.seo_in_index = b.seo_in_index
    and a.column_name = b.column_name
    where a.seo_in_index = 1
    and a.index_name <> b.index_name;
```

##### 通过工具 pt-duplicate-key-checker

```
    pt-duplicate-key-checker \
    -h www.myapp.com
    -uroot \
    -p
```

##### 通过工具 pt-index-usage

目前 MySQL 中没有记录索引的使用情况，只能通过慢查询日志配合 `pt-index-usage` 工具来分析索引使用情况。

```
    pt-index-usage \
    -uroot -p ''\
    mysql-slow.log
```

> PerconMySQL 和 MariaDB 中可以通过 INDEX_STATISTICS 表来查看哪些索引未使用。

### 删除无用索引

```
    alter table tb drop index `useless`;
```

## 查询优化

### 如何分析 SQL 查询？

#### 使用 explain

查询 SQL 的执行计划。使用方式：

```
    explain select * from tb_name;
```

#### explain 返回列解释

- **table：** 显示这一行的数据是关于哪张表的
- **type：** 显示连接使用了何种类型。从最好到最差的连接类型为：const => eq_reg => ref => range => index => all
- **possible_keys：** 显示可能应用在这张表中的索引，为空则可能没有索引。
- **key：** 实际使用的索引，为 NULL，则没有使用索引。
- **key_len：** 使用的索引的长度。在不损失精确性的情况下，长度越短越好。
- **ref：** 显示索引的哪一列被使用了，如果可能的话，是一个常数。
- **rows：** MySQL 认为必须检查的，用来返回请求数据的行数。
- **extra：** 存放额外返回结果。当返回值为以下 2 种时，表示查询需要优化：
  - **Using filesort：** MySQL 需要进行额外的步骤来发现对如何返回的行排序。它根据连接类型，以及存储排序键值和匹配条件的全部行的行指针来排序全部行。
  - **Using temporary：** MySQL 需要创建一个临时表来存储结果，这通常发生在对不同的列集进行 order by 上，而不是 group by 。


### 慢查询

#### 慢查询日志的设置

MySQL 慢查询日志监控了有效率问题的 SQL。下面是对慢查询日志的设置：

```
    show variables like 'slow_query_log';
    
    -- 开启慢查询
    set global slow_query_log = on;
    
    -- 指定了慢查询日 志文件的存储路径
    set global slow_query_log_file = '/data/mysql/log/mysql-slow.log';
    
    -- 设置是否开启把没有使用索引的 SQL 记录到慢查询日志中，由于通常情况下对数据库的优化主要就是优化索引和查询的 使用方式，因此这里必须开启，方便分析
    set global log_queries_not_using_indexes = on;
    
    -- 设置把超过多少秒的慢查询记录到日志中，实际应用中一般 100 ms
    set global long_query_time = 1;
```

设置好后，可以进行查询测试，然后观察慢日志的监控：

```
    tail -10 /var/log/mysql/mysql-slow.log
```

#### 慢查询日志包含的内容

- 执行 SQL 的主机信息
- SQL 的执行信息
- SQL 的执行时间
- SQL 的内容

#### 慢查询日志分析工具

- mysqldumpslow：安装 MySQL 之后自带的分析工具。

```
    # 查询服务器中查询最慢的 3 条 SQL
    mysqldumpslow -t 3 /var/log/mysql/mysql-slow.log | more    # more 是滚动符
```

- pt-query-digest

```
    # 下载安装
    apt-get install percona-tookit -y
    pt-query-digest --help
    pt-query-digest /var/log/mysql/mysql-slow.log | more
    
    # 输出到文件
    pt-query-digest slow-log > slow_log.report
    
    # 输出到数据库
    pt-query-digest slow.log -review \
    h=127.0.0.1, D=test, p=root, P=3306, u=root, t=query_review \
    --create-reviewtable \
    --review-history t = hostname_slow
```

在 pt-query-digest 的分析结果中，主要关注：

- 查询次数多且每次查询占用时间长的 SQL：通常为 pt-query-digest 分析的前几个查询。
- IO 大的 SQL：注意 pt-query-digest 分析中的 ‘Rows examine’ 项目。
- 未命中的 SQL：注意 pt-query-digest 分析结果中的 ‘Rows examine’ 和 ‘Rows Send’ 的对比。

### 常见优化场景

#### count()

如何同时查询到两个条件下的记录图条数？

- 错误的查询：

```
    -- 无法分开计算
    select count(k1='v1' or k2='v2') from tb;
    
    -- 逻辑错误
    select count(*) from tb where k1='v1' and k2='v2';
```

- 正确的查询：

```
    select count(k1='v1' or null) as 'res1'
    , (k2='v2' or null) as 'res2' from tb;
```

#### count(*) 和 count(字段)

```
    create table tb(id int);
    insert into tb values (1), (2), (null);
    
    select count(*), count(id) from tb;
    -- count(*): 3
    -- count(id): 2
```

可见，`count(*)` 会返回含 NULL 的记录，而 count(字段) 的结果是不包括 null 的结果。

#### max()

对 max() 的优化可以体现在创建索引的基础上。

举例说明，为 max() 函数的求值字段创建覆盖索引前后的对比：

```
    explian select max(age) from user\G
    create idx_age on user(age);
    explian select max(age) from user\G
```

对比创建索引前后，明显的差别是，Extra 字段中从空到 `‘Select tables optimized away’`，type 从 all 到 null，表明创建索引后查询操作只是通过索引来得到结果的，而没有进行表的操作，从而减少了 `I/O` 操作。

> 根据索引来查询的效率是比较恒定的。

#### 子查询

通常情况下，需要把子查询优化为 join 查询，但是在优化时要注意关联键是否有一对多的关系，**要注意重复记录。**

**举例说明:**

```
    create table t1 (id int);
    create table t2 (id int);
    
    -- 子查询
    select * from t1 where id in (select id from t2);
    
    -- join 查询
    select t1.id from t1 join t2 on t1.id = t2.id;
```

当关联的字段有一对多的关系时，比如：

```
    insert into t1 values(1);
    insert into t2 values(1), (1);
```

这时 t1 的 id = 1 的情况下，使用子查询是由一条记录，而使用 join 查询有 2 条记录。因此，如果不想要重复记录，则需要使用 distinct 去重。

```
    select distinct t1.id from t1 join t2 on t1.id = t2.id;
```

注意：on 关联的左右字段不是一对一时，是一个比较出错的地方。

#### group by

如果 explain 的结果 Extra 中使用了临时表和文件排序等 I/O 操作，则有必要进行这种优化( 也是使用索引的思想 )，尽量避免使用临时表和文件排序，group by 什么字段就 using 什么字段。

当然，也可以增加一些过滤条件实现同样的效果和性能，不过需要注意在子查询中增加过滤条件。

使用 sakila 数据库举例说明：
    
```
    -- 优化前
    explain
    select actor.first_name, actor.last_name, count(*)
    from sakila.film_actor
    inner join sakila.actor using(actor_id)
    group by film_actor.actor_id;
    
    -- 优化后( 减少连接表中不必要的字段 )
    explain
    select actor.first_name, actor.last_name, c.cnt
    from sakila.film_actor
    inner join (
      select actor_id, count(*) as cnt
      from sakila.film_acotr
      group by actor_id
    ) as c using(actor_id)
```

#### limit

limit 常用语分页处理，和 order by 从句共同使用，因此经常使用 `Filesorts` 这样的，会造成大量的 `I/O` 操作的问题。

**举例说明：**

- 优化前

```
    select film_id, description
    from sakila.film
    order by title
    limit 50, 5;
```

- 优化后

```
    -- 1. 使用有索引的列或者主键进行 order by 操作
    select film_id, description
    from sakila.film
    order by film_id
    limit 50, 5;
    
    -- 2. 记录上次返回的主键, 下次查询时使用主键过滤
    select film_id, description
    from sakila.film
    where film_id > 55
    and film_id <= 60
    order by film_id
    limit 1, 5;
```

使用第二种优化方式时，可以避免数据量过大时扫描过多的记录。

不过，有个前提条件就是主键要是连续增长的，如果不是连续，那么很可能列出的记录数量不符合预期的情况。如果不满足这个前提条件，可以新增一个附加列（自增，并建立好索引），然后根据这个附加列查找。


# 其他

## MySQL 提供的测试数据库

- 查看 MySQL 版本是否大于 5.5：`select @@version`;（版本不同优化器有差别）
- 下载、导入 <u>[sakila](http://dev.mysql.com/doc/index-other.html)</u> 数据库
    
```
    unzip sakila.zip
    mysql -uroot -p
    source sakila-db/sakila-schema.sql
    source sakila-db/sakila-data.sql
    use sakila;
    show tables;    -- 23 rows
```

## 可能用到的 Linux 命令

```
    find / -name log
    tail -50 /path/to/file.log
    mysqldumpslow -h
```


# 参考

- [SAN vs. NAS: What Is the Difference?](https://www.lifewire.com/san-vs-nas-818005)
- [MySQL 优化](http://blog.cjli.info/2016/02/04/MySQL-Optimization/)