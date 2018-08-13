---
title: MySQL 查看表结构
date: 2018-07-26 09:41:59
categories: MySQL
tags: [MySQL]
---

# 简单描述表结构, 字段类型

```
    desc table_name;
```

<!-- more -->

**例如:**

```
    MariaDB [skb]> desc users;
    +----------------+------------------+------+-----+---------+----------------+
    | Field          | Type             | Null | Key | Default | Extra          |
    +----------------+------------------+------+-----+---------+----------------+
    | id             | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
    | name           | varchar(191)     | NO   |     | NULL    |                |
    | email          | varchar(191)     | NO   | UNI | NULL    |                |
    | password       | varchar(191)     | NO   |     | NULL    |                |
    | homepage       | varchar(191)     | NO   |     | NULL    |                |
    | remember_token | varchar(100)     | YES  |     | NULL    |                |
    | created_at     | timestamp        | YES  |     | NULL    |                |
    | updated_at     | timestamp        | YES  |     | NULL    |                |
    +----------------+------------------+------+-----+---------+----------------+
```

# 查看表中列的注释信息

```
    select * from information_schema.columns 
    where table_schema = 'db'      # 表所在数据库
    and table_name = 'table_name'; # 要查询的表
```

**例如:**

```
    select * from information_schema.columns  where table_schema = 'skb' and table_name = 'skb_users'; 
```

# 只查询列名和注释

```
    select column_name, column_comment from information_schema.columns 
    where table_schema = 'db'      # 表所在数据库
    and table_name = 'table_name'; # 要查询的表
```

**例如:**

```
    MariaDB [skb]> select column_name, column_comment from information_schema.columns  where table_schema = 'skb' and table_name = 'skb_users'; 
    +-------------+-------------------------------------------------------------+
    | column_name | column_comment                                              |
    +-------------+-------------------------------------------------------------+
    | id          |                                                             |
    | username    | 用户名                                                      |
    | password    | 密码                                                        |
    | openid      | 微信ID                                                      |
    | nickname    | 微信昵称,用户昵称                                           |
    | avatar      | 微信头像,用户头像                                           |
    | mobile      | 微信手机号,用户手机号                                       |
    | is_del      | 1表示删除,0表示未删除                                       |
    | role        | 1表示仅是用户,2表示仅是师傅,3表示两者皆是                   |
    | created_at  |                                                             |
    | updated_at  |                                                             |
    | deleted_at  |                                                             |
    | money       | 余额                                                        |
    +-------------+-------------------------------------------------------------+
    13 rows in set (0.00 sec)
```

# 查看表的注释

```
    select table_name, table_comment from information_schema.tables 
    where table_schema = 'db'      # 表所在数据库
    and table_name = 'table_name'; # 要查询的表
```

**例如:**

```
    MariaDB [skb]> select table_name, table_comment from information_schema.tables  where table_schema = 'skb' and table_name = 'skb_users'; 
    +------------+---------------+
    | table_name | table_comment |
    +------------+---------------+
    | skb_users  |               |
    +------------+---------------+
    1 row in set (0.00 sec)
```

# 查看表生成的`DDL`

```
    show create table table_name;
    
    # 更美观的显示
    show create table table_name \G;
```

**例如:**

```
    MariaDB [skb]> show create table skb_users\G;
    *************************** 1. row ***************************
           Table: skb_users
    Create Table: CREATE TABLE `skb_users` (
      `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
      `username` varchar(191) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '用户名',
      `password` varchar(191) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '密码',
      `openid` varchar(191) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '微信ID',
      `nickname` varchar(191) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '微信昵称,用户昵称',
      `avatar` varchar(191) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '微信头像,用户头像',
      `mobile` char(191) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '微信手机号,用户手机号',
      `is_del` tinyint(4) DEFAULT '0' COMMENT '1表示删除,0表示未删除',
      `role` tinyint(4) NOT NULL DEFAULT '1' COMMENT '1表示仅是用户,2表示仅是师傅,3表示两者皆是',
      `created_at` timestamp NULL DEFAULT NULL,
      `updated_at` timestamp NULL DEFAULT NULL,
      `deleted_at` timestamp NULL DEFAULT NULL,
      `money` double(8,2) NOT NULL COMMENT '余额',
      PRIMARY KEY (`id`)
    ) ENGINE=MyISAM AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
    1 row in set (0.00 sec)
    
    ERROR: No query specified
```