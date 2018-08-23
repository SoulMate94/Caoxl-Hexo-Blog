---
title: MongoDB 可视化管理工具
date: 2018-08-16 15:17:15
categories: NoSQL
tags: [NoSQL, MongoDB]
---

> `MongoDB Compass`是`MongoDB`官网提供的一个集创建数据库、管理集合和文档、运行临时查询、评估和优化查询、性能图表、构建地理查询等功能为一体的`MongoDB`可视化管理工具。

<!-- more -->

# `MongoDB Compass` 的安装

## `MongoDB` 下载

> [MongoDB Download Center](https://www.mongodb.com/download-center#community)

具体步骤详见: [Windows 平台安装 MongoDB](http://www.runoob.com/mongodb/mongodb-window-install.html)

这里就不多做赘述了~


# `MongoDB Compass` 的使用

## 1、创建MongoDB数据库连接

![创建MongoDB数据库连接](https://caoxl.com/images/mongo/mongo-connection.png)

分别输入相应的`Hostname`和`Port`，如果没有用户认证，`Authentication`就默认为空。添加完后，点击`CONNECT`即可连接。


## 2、创建数据库

连接`MongDB`数据库后，可以点击`create database`创建一个数据库。我这里创建了`test_mongo`数据库的同时也创建`test_mongo`集合。

![创建MongoDB数据库连接](https://caoxl.com/images/mongo/mongo-create-database.png)


## 3、集合管理

### 插入文档

在`test_mongo`集合的`Documents`页签下，点击`INSERT DOCUMENT`插入文档。

![插入文档](https://caoxl.com/images/mongo/mongo-insert-doc.png)

文档查看有两种方式一个是list（列表）方式，一个是table（表格）方式。

![list（列表）方式](https://caoxl.com/images/mongo/mongo-view-list.png)
![table（表格）方式](https://caoxl.com/images/mongo/mongo-view-table.png)

### 执行文档查询

在`FILTER`行输入查询条件后，点击`FIND`，即可执行查询。

![执行文档查询](https://caoxl.com/images/mongo/mongo-find.png)


## 4、集合


![集合](https://caoxl.com/images/mongo/mongo-aggregations.png)


## 5、解释执行计划

在`Explain Plan`页签中，可以在`FILTER中`输入相关的查询语句后，点击`EXPLAIN`查看该语句解释执行计划。这个解释执行计划跟关系型数据库的`SQL`执行计划，有点类似。

![解释执行计划](https://caoxl.com/images/mongo/mongo-explain-plan.png)

## 6、索引

在`Indexes`页签可以为集合创建索引及查看该集合下的索引详细信息。

![索引](https://caoxl.com/images/mongo/mongo-indexes.png)


# 附录

- [MongoDB 简介](http://blog.caoxl.com/2018/01/22/MongoDB-Intro/)
- [RUNOOB.com MongoDB](http://www.runoob.com/mongodb/nosql.html)