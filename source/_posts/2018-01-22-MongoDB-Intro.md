---
title: MongoDB 简介
date: 2018-01-22 18:03:16
categories: MySQL
tags: [Database, MongoDB]
---

> MongoDB  是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。

# 简介

## 关键词

- 开源
- NoSQL
- 免费
- 良好的技术支持

因此，国内外许多互联网公司都已经采用了 MongoDB。

<!--more-->

## 概念

### 数据库

#### 什么是数据库？

- 有组织地存放数据
- 按照不同的需求进行查询

不同数据库的区别就是，**对存放数据的组织不同，同时也提供了不同的查询方式根据需求选择不同的数据库**。

#### 数据库分类

##### 关系型

支持 SQL 语言的数据库，`MySQL`，`Oracle` 等。具有如下特点：

- 实时一致性
- 事务
- 多表联合查询

##### NoSql

不支持 SQL 语言的数据库，`Redis`, `MongoDB` 等。特点如下:

- 简单便捷
- 方便扩展
- 更好的性能


#### MongoDB

##### Why MongoDB ?

- 无数据结构的限制
  - 没有表结构的概念，每条记录可以有完全不同的结构（JSON）
  - 业务开发方便快捷
  - 无需事先定义存储结构
  关系型数据库需要先定义好表结构才能使用。MongoDB 的这种特点非常适合快捷开发和多变的业务需求。
  
- 完全的索引支持
  - 类似 `redis` 的键值存储
  - 类似 `hbase` 的单索引，二级索引需要自己实现
  - 支持：单键索引、多键索引、数组索引、全文索引、地理位置索引。
  
 > 全文索引还不支持中文。
 > 
 > MongoDB 被称为最接近 SQL 的 NoSQL 数据库。
 
- 方便的冗余与扩展
  - 复制集保证数据安全
  - 分片扩展数据规模

- 良好的支持
  - 完善的文档
  - 齐全的驱动支持

##### 学习方向

- mongo
- 索引
- 集合
- 复制集
- 分片
- 数据均衡

## 学习方向

### 基本使用

- 最基本的文档读写、增删改
- 各种不同类型的索引创建与使用
- 复杂的聚合查询
- 对数据集进行分片，在不同的分片间维持数据均衡
- 数据备份与恢复
- 数据迁移

### 部署数据库服务

- 搭建简单的单机服务
- 搭建具有冗余容错功能的复制集
- 搭建大规模数据集群
- 完成集群的自动部署

### 简单运维

- 部署 MongoDB 集群
- 处理常见故障
  - 单节点失效后如何恢复工作
  - 数据库意外停止如何恢复数据
  - 数据库发生拒绝服务时如何排查原因
  - 数据库磁盘快满时如何处理
  
# 安装和配置

## 下载安装

> 软件版本命名格式：a.b.c
>
> - a 是大版本，重大更新
> - b 是稳定版和非稳定版
> - c 是小版本，性能提升和 bug 修复

```
    # download from mongodb website
    tar xzvf mongodb.xxx.tgz
    
    # clone from github
    apt-get install zip
    unzip mongodb.zip
    
    cd /mongodb.xxx/
    apt-get install scons    # scons 是替代 make 的另一种编译方式
    scons all -j 4    # -j 选项后面跟上 CPU 核数以加速编译过程
```

如果多次编译都不能通过，也可以直接到官网去下载编译好的可执行二进制文件。

如果 Linux 是 `32` 位的，这时为了方便建议直接使用包管理器安装：`apt-get install mongodb`。

> Starting in MongoDB 3.2, 32-bit binaries are deprecated and willbe unavailable in future releases.
> 
> 32 bit builds are limited to less than 2GB of data (or less with ­­journal).

## 搭建简单的 MongoDB 服务器

```
    mkdir mongo_server cd mongo_server mkdir data
    mkdir log
    mkdir conf-f
    mkdir bin
    cp /path/to/mongo-xxx/mongod ./bin cd conf
    
    vi mongod.conf
      port = 12345
      dbpath = data # 存储数据的路径
      logpath = log/mongod.log
      fork = true # true 表示启动后台进程
      
    cd ..
    ./bin/mongod -f conf/mongod.conf cd data/
    ls
    tail ../log/mongod.log
```

运行 MongoDB 服务器：

```
    mongod -f /etc/mongod.conf
```

如果不修改默认配置文件，那么执行 `mongo` 就可以直接进入 `MongoDB` 服务器了，进入后默认选择的数据库是 `test`。

但是如果修改了默认配置，就需要指定到正确的配置文件，比如修改了端口为 12345，那么在就需要带上新指定的端口号才能进入：

```
    mongo localhost:12345
```

连接 MongoDB 服务器主要有 2 种方式：

- `mongodb` 自带的客户端
- `mongodb` 为各种编程语言提供的驱动程序

```
    /path/to/mongo ip:port/dbname    # 默认进入不需要认证
    
    # 关闭 Mongodb 服务器
    use admin
    db.shutdownServer()
```

> Kali 32 树莓派 2 上的 MongoDB 使用 `apt-get` 安装好之后，配置文件 默认在 `/etc/mongod.conf`，日志文件在 ：`/var/log/mongodb/mongodb.log`。

# 基本使用

## 创建数据库

```
    show dbs
    
    use db_name
```

使用 `use` 切换数据库的时候可以没有创建该数据库，`mongodb` 会在合适的时候自动创建数据库。

## 删除数据库

选择了某个数据库后，就可以进行如下操作：

```
    db.dropDatabase()
```

关系型数据库中的表在 mongodb 中被称为「集合」。（show collections <=> show tables）

**注意：**

- 插入数据的时候注意使用 JSON 格式的数据结构，使用可嵌套的键值对存储数据
- 字符串需要用引号括起来，插入数据后会自动创建数据库和 system.indexes 文件

## 查找数据

```
    db.dbname_collectionname.find( {x:1} ) db.dbname_collectionname.findOne()
```

会看到一个 `_id` 字段的键值对，这是 `mongodb` 自动生成的 `id`，在全局范围内不会重复。


## 插入数据

```
    db.dbname_collectionname1.insert({ x:1 }) db.dbname_collectionname2.insert({ 'gender':'male' })
```

也可以使用 JS 语法在循环中插入数据：

```
    for ( i=3; i<100;++i ) {
      db.collectionname.insert( {y:i} )
    }
```

find() 后也可以跟上一些方法和条件，比如：

- count() 计数
- skip() 跳过
- limit() 限制条数
- sort() 排序

```
    db.collectionname.find().skip(5).limit(3).sort({x:1})
```

方法后面必须有括号 `()`。


## 创建集合

```
    db.createCollection( 'collection_name' )
```

## 更新数据

```
    db.collectionname.update( { x:1}, {x:222} )
```

### 部分更新

```
    db.collectionname.insert( {x:1}, {y:22}, {z:333} )
    
    # 仅更新 y 字段
    db.collectionname.update( {z:333}, {$set: {y:2222}} )   // 这里的 z:333 必须和插入的数据值相同否则更新无效
```

### 更新不存在的数据

设置如果更新的数据不存在就写入这条数据：

```
    db.collectionname.update( {x:1}, {x:1111}, true )
```

如果不设置为 true 那么更新不存在的数据的时候是不会有任何影响的。

### 更新多条数据

为了防止 update 误操作，mongodb 默认 update 只会更新第一条 找到的数据。

如果要多记录更新，需要指定 `update()` 的第二个参数为 `set` 操 作，并且第四个参数为 `true`。

```
    db.collectionname.update( {test:1}, {$set:{test:1024}}, false, true )
```

## 删除数据

```
    db.collectionname.remove() // 可以不指定参数
```

`remove()` 默认会删除所有指定的数据记录，比如：

```
    db.test.remove( {test:1024} ) # 会删除所有 test 为 1024 的记录
```

### 删除集合(表)

```
    db.collection.drop()
```

## 创建集合的索引

在数据量较小的时候查询速度还可以，但是当数据记录量较大时，没有相关字段的索引，查询的速度将变得非常慢，甚至没有结果。

### 查看索引

```
    db.collection.getIndexes()
```

### 创建索引

```
    db.collection.ensureIndex({ key:1 })
```

其中，键 `key` 代表的是排序方向，`1` 为正向排序，`-1` 为逆向排序。


### 关于索引和性能

如果记录太多，创建索引的时间会比较长。

如果系统负载较重，且有很多已经存在的文档，不能使用 `ensureIndex()` 来动态创建索引，因为这样会极大影响数据库的性能，这种情况需要事先预料，并创建好相关索引。

上面创建好 `key` 的索引后，再次使用查询 `key` 的时候性能会大幅度提高。对于常见的查询，对其字段创建索引是非常有必要的。

由于索引是在插入数据后再次构建，所以在创建索引的时候可能会对数据库写入操作造成影响，但是为了查询的高效性，这点影响是可以接受的。


# 常见索引

## `_id` 索引

- `_id` 是绝大多数集合默认建立的索引
- 对于每个插入的数据，mongodb 都会自动生成一条唯一的 `_id` 字段

## 单键索引

- 单键索引是最普通的索引
- 与 `_id` 不同，单键索引不会自动创建

比如，一条记录如果为：`{x:1,y:2,z:3}`。

在 x 字段上创建索引后，就可以使用 x 字段为条件进行查询。索引可以重复创建，对于已经存在的索引，再次创建时会直接返回成功。

## 多键索引

多键索引与单键索引的创建形式相同，区别在于字段的数量。

- **单键索引：** 值为一个单一的值，例如字符串，数字或者日期。
- **多键索引：** 值具有多个记录，例如数组。

比如，`db.test.insert({x:[1,2,3,4,5]})`，执行该命令后，mongDB 会为 x 字段创建多键索引。

## 复合索引

当我们的查询条件不止一个时，就需要建立复合索引。

```
    db.collection.ensureIndex({x:1, y:1})
    db.test.ensureIndex({x:1,y:2})
    db.test.find({x:1,y:2})
```

## 过期索引

是在一段时间后会过期的索引。具有如下特点：

- 在索引过期后，相应的数据会被删除。
- 适合存储一段时间后会失效的数据，比如用户的登录信息，存储的日志等。

建立方法：

```
    db.collection.ensureIndex({time:1},{expireAfterSeconds:10})
    
    db.test.ensureIndex( {time:1}, {expireAfterSeconds:30} ) db.test.insert( {time:new Date()} )
    db.test.find()
    
    # 30 秒后再次执行
    db.test.find()
```

可以发现，即使设置了 `30` 秒的生存时间，也会在 `60` 秒之后才被删除。

这是因为删除过程是由后台程序每 `60` 秒执行一次，而且删除本身也需要一些时间，所以存在误差。

### 限制

- 存储在过期索引字段的值必须是指定的时间类型。
> 必须是 ISODate 或 ISODate 数组，不能使用时间戳，否则不能被自动删除。
- 如果制定了 ISODate 数组，则按照最小的时间进行删除。（最小时间删除原则）
- 过期索引不能是复合索引
- 删除时间不是精确时间


## 全文索引

对字符串与字符串数组创建全文可搜索的索引。

**举例说明：**

```
    {author:'', title:'', article:''}
```

则为其建立全文索引的方法：

```
    db.articles.ensureIndex({key:'text'})
    db.articles.ensureIndex({key_1:'text', key_2:'text'})
    db.articles.ensureIndex({'$**':'text'})
```

最后一条方法代表的是为该集合的所有字段建立一个全文索引。

然后就可以使用全文索引进行查询了：

```
    db.articles.find({$text {$search: 'coffee'}})
    db.articles.find({$text {$search: 'a b c'}})
    db.articles.find({$text {$search: 'a b -c'}})
    db.articles.find({$text {$search: '\'a\' b -c'}})
```

可见使用全文索引查询不用再指定字段名。

`MongoDB` 中每个数据集合只允许创建一个全文索引。

`a b c` 之间默认是’或’关系，如果需要`’与’`关系的查找，则需要将每个独立 的关键字用引号括起来。当然，需要注意转义。

`-c` 代表不包含 `c`。

### 全文索引相似度

通过 `$meta` 操作符：`{score: {$meta: 'textScore'}}`

写在查询条件后面可以返回结果的相似度。与 `sort` 一起使用，可以达到很好的实用效果。

`score` 是一个数字，表征的是查询结果的相似度。

也可以在查询结果后面带上排序：

```
    db.articles.find( {$text: {$search:"xxxx aaaa"}}, {score:{$meta:"textScore"}} ).sort({score:{$meta:"textScore"}})
```

### 限制

- 每次查询，只能指定一个 `$text` 查询
- `$text` 查询不能出现在 `$nor` 查询中
- 查询中如果包含了 `$text`，`hint` 不再起作用（`hint` 用来手工强制指定索引来测试索引的性能）
- 暂不支持中文。


## 地理位置索引

将一些点的位置存储在 MongoDB 中，创建索引后，可以按照位置来查找其他点

### 分类

- **2d 索引：** 用于存储和查找平面上的点
- **2dsphere 索引：** 用于存储和查找球面上的点

### 查找方式

- 查找距离某个点一定距离内的点
- 查找包含在某区域内的点

### 2D 索引

平面地理位置索引。

```
    db.collection.ensureIndex({w: ' 2d'})
```

- 位置表示方式：经纬度。
- 取值范围：经度[-180, 180]；纬度 [-90, 90]
- 查询方式
  - `$near：`查询距离某个点最近的点
  - `$geoWithin：`查询某个形状内的点

```
    db.location.ensureIndex( {"w":"2d"} )
    db.location.insert( {w:[180, 100]} ) // 这样的点虽然能被创建但是会带来不可预知的错误，因此不要插入这样的点
    db.location.remove( {w:[180, 100]} ) // 删除不合法的地点
    db.location.insert( {w:[180, 10]} )
    db.location.find( {w:{$near:[1,1]}}, $maxDistance:10 ) 持 $minDistance
```

默认会查询 100 条结果。

- 形状的表示
  - `$box：`矩形
  - `$center：`圆形
  - `$polygon：`多边形
  
```
    db.location.find( {w:{$geoWithin:{$box:[0,0], [3,3]}}} )
    和右边界
    
    // 参数是 2 个边界:左边界 // 圆心 + 半径
    db.location.find( {w:{$geoWithin:{$center:[0,0], 5}}} )
    db.location.find( {w:{$geoWithin:{$polygon:[0,0], [3,3], [2, 4], [1, 5]}}} ) // 多个点组成的多边形
```

- geoNear 查询

geoNear 使用 runCommand 命令进行使用，常用使用如下：

```
    db.runCommand(
    { geoNear: "location", near:[1,2], maxDistance:10, num:1 } )
```

`minDistance` 对 `2D` 索引无效但是对 `2dsphere` 有效，`num` 限制返回的数据数目。

可见，`geoNear` 比 简单的 `near` 能够返回更多的数据


### 2dsphere

球面地理位置索引。创建方式：

```
    db.collection.ensureIndex({w:'2dsphere'})
```

位置表示方式：getJSON 描述一个点，一条直线，多边形等形状。
    
```
    {type: ' ', coordinates:[<coordinates>]}
```

查询方式和 2d 索引查询方式类似，支持 `$minDistance` 与 `$maxDistance`。

`2dsphere` 还可以查询几个多边形的相交点。


## 索引构建情况分析

- **索引好处：** 加快索引相关的查询。
- **索引坏处：** 增加磁盘空间消耗，降低写入性能。

### 如何评判当前索引构建情况

- mongostat 工具

查看 mongodb 运行状态：

```
    ./mongostat -- help
    
    mongostat -h 127.0.0.1:12345    // idx miss
```

对于性能我们主要关心的是 `qr` 和 `qw`，分别代表的是读写队列。而 `idx miss` 代表的是查询的时候没有使用到索引的百分比，越高则越需要作相应的处理。

- profile 集合

```
    db.getProfilingStatus()
    db.getProfilingLevel() // 结果为 0 则代表 profile 是关闭的;为 1 的时候会记录所有 slowms 的操作
    db.setProfilingLevel( 2 ) // 2 代表记录所有操作
```

开启 profile 后，数据库中就会创建一个 system.profile 的集合。

然后查询：

```
    db.system.profile.find().sort({$natural:-1}).limit(1)
```

> 注意：profile 的使用场景是上线前的测试阶段以及刚上线时的观察阶段，仅仅作为了解 系统状态的工具，因为当数据较多的时候它会占用很多资源，从而影响 MongoDB 的性能。

- 日志

```
    vi /etc/mongod.conf
```

可以用 `verbose = vvvvv` 来配置日志的详细程度，v 越多记录的日志越详细。

- explain 分析

```
    db.test.find( {x:1} ).explain()
```

会得到一次操作中使用到的参数信息，比如是否使用索引，耗时等。

# 索引属性

即创建索引时的格式：

```
    db.collection.ensureIndex({param}, {param})
```

其中，第二个参数变色索引的属性。

比较重要的属性有：

- 名字

`name` 指定：`db.collection.ensureIndex({}, {name: ' '})。`

MongoDB 对字段名有长度限制，目前是 125 字节，可以为复合索引设置 name 简化字段命名。

```
    db.test.ensureIndex({x:1, y:1, z:1, m:1}, {name:"normal_index"})
```

然后就可以使用设置的名字来删除复合索引：

```
    db.test.dropIndex( "normal_index" )
```

- 唯一性

`unique` 指定：`db.collection.ensureIndex({}, {unique:true/false})`。

利用 unique 可以实现 MySQL 中如果存在则不插入如果不存在则插入的功能。

- 稀疏性

`sparse` 指定：`db.collection.ensureIndex({}, {sparse: true/false})`。

`mongoDB` 默认创建的索引都是不稀疏的。稀疏性的不同代表了 `MongoDB` 处理索引中存在但是文档中不存在的字段时，会采取怎样的方法。

设置稀疏性索引不必为不存在数据的，但是被创建了索引的字段上建立索引，可以减少磁盘占用，提高插入速度。

稀疏索引的使用可能会带来一些隐患，因此往往配合 `exists` 操作符。

```
    db.test.insert({"m":1})
    db.test.insert({"n":2})
    db.test.find({m:{$exists:true}}) // 为 true 代表查找的是存在 m 的记录 db.test.ensureIndex({m:1},{sparse:true}) // 在 m 字段上创建稀疏索引
     
    db.test.find({m:{exists:false}}) // 查找不存在 m 的记录
    db.test.find({m:{exists:false}}).hint( "m_1" ) // 强制使用稀疏索引则不会再找到 n 所在的记录
```

由于 `MongoDB` 在选取索引时如果发现是在稀疏索引上查找不存在的记录的时候，不会使用创建过的稀疏索引，因此，这里仍然会看到结果为 n 的记录。

因此，不能在稀疏索引上查找不存在的记录。

- 是否定时删除

`expireAfrerSeconds` 指定。

# 安全相关

## 概览

- 最安全的是物理隔离，但是不现实
- 网络隔离其次
- 防火墙、IP 白名单其次
- 用户名密码最后

## 权限设置

- `auth` 开启
- `keyfile` 开启

默认情况 `MongoDB` 没有开启权限验证
    
```
    vi /etc/mongod.conf
    
    	auth = true
    	
    ps -ef | grep mongod | grep 12345
    kill pid
    
    mongod -f /etc/mongod.conf
```

此时查看日志就能够发现鉴权已开启：`/auth`。

但仅仅开启鉴权，进入 `MongoDB` 仍然不需要用户名和密码，因为还需要为 `MongoDB` 服务器创建用户名和密码，只有创建用户之后才能激活鉴权机制。

## 创建用户
   
MongoDB 内建用户类型有：

- read
- readWrite
- dbAdmin
- dbOwner
- userAdmin

除了内建类型，还可以自定义角色类型。

```
    db.createUser({user:"li", pwd:"enter", customData:"示例",roles: [{role:"userAdmin", db:"admin"}, {role:"read",db:"test"}]})
```

创建成功后，依然可以无密码登录，但是在执行相应的操作时候会提示没有操作权限这时需要使用用户名和密码登录
    
```
    mongo -ip:port -u root -p pwd
```

此外，即使使用了用户名和密码登录，在操作有些集合的时候也不一定一定能操作成功，这要根据对具体集合的权限分配。

## 用户角色

- 数据库角色
  - read
  - readWrite
  - dbAdmin
  - dbOwner
  - userAdmin
  
- 集群角色
  - clusterAdmin
  - clusterManger
  
- 备份角色
  - backup
  - restore

- 其他特殊权限
  - DBAdminAnyDatabase
  
此外，还有 2 种不对外开放的权限：`root` 和 `__system`。

```
    {
      _id : "myapp.appUser",
      
      role : "appUser",
      
      db : "myapp",
      
      privileges : [
        { resource : {db: "myApp", collection : " "},
          actions : ["find", "createCollection", "dbStats", "collStats"]},
        { resource : {db: "myApp", collection : " logs "},
          actions : ["insert"]},
        { resource : {db: "myApp", collection : " data "},
          actions : ["insert", "update", "remove", "compact"]},
        { resource : {db: "myApp", collection : " system.indexes "},
          actions : ["find"]},
        { resource : {db: "myApp", collection : " system.namespaces "},
          actions : ["find"]}
      ],
      
      roles : [],
    }
```

这是使用 `createRole()` 方法对自定义权限的配置，这样的方式更灵活，也更 方便管理员掌控

# 参考

- [SQL vs Nosql ：优劣大对比](http://www.d1net.com/datacenter/tech/256374.html)
- [8 种 NoSQL 数据库系统对比](http://blog.jobbole.com/1344/)