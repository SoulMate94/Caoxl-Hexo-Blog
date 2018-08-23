---
title: Redis 使用日志
date: 2018-08-20 15:06:31
categories: 缓存
tags: [Cache, Redis, RTFM]
---

> `Redis` 在开发、测试、正式环境使用日志，不定时更新。

<!-- more -->

说明：本文旨在梳理 Redis 这门技术的学习思路，实际使用必须 <u>[RTFM](https://zh.wikipedia.org/wiki/RTFM)</u>: [Redis 命令参考](Redis 命令参考) / [英文版](https://redis.io/commands/)

# 简介

## 定义

`Redis`（`REmote Dictionary Server`，远程字典服务器） 是一个开源、高性能、基于键值对的缓存与存储系统，允许其他应用以字典（KV）形式通过 TCP 读写字典中的内容。

## Redis 的优点

- 更符合程序员思维

Redis 字典结构的存储方式和对多种键值数据类型的支持，使得开发者可以将程序中的数据直接映射到 Redis 中，因为数据在 Redis 的存储形式和在程序中的存储方式非常相似。

- 各种数据类型有丰富的操作

Redis 中每种数据结构都具有很多非常方便的接口，可以实现部分关系型数据库实现起来性能不高且较为繁琐的操作。

- 支持持久化的内存存储，快速可靠

Redis 的所有数据都存储在内存中，因此在性能上比其他基于硬盘的数据库高。此外，Redis 也会将内存中的数据异步地写入到硬盘中，同时不影响继续提供服务。

Redis 支持 `AOF` 和 `RDB` 两种持久化方式。

> RDB 将数据库的快照（snapshot）以二进制的方式保存到磁盘中。
> AOF 则以协议文本的方式，将所有对数据库进行过写入的命令（及其参数）记录到 AOF 文件，以此达到记录数据库状态的目的
> > https://redisbook.readthedocs.io/en/latest/internal/aof.html

- 应用广

Redis 是作为数据库开发的，但是由于其功能丰富，也被广泛应用到缓存、队列系统中。

- 使用简单

- Redis 在各种数据类型上均提供了很直观的调用方式。

- 开源，代码量少可定制

Redis 源代码只有 3W+ 行，可以为不同的项目定制更适合的 Redis 版本。

- 轻量级

一个空 Redis 实例只会占用 1M 左右内存，因此不用担心启动多个 Redis 实例会占用额外的内存。

- 支持集群

支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。

## Redis 的缺点

- 由于是内存数据库，所以，单台机器，存储的数据量，跟机器本身的内存大小。虽然 Redis 本身有key过期策略，但是还是需要提前预估和节约内存。如果内存增长过快，需要定期删除数据。

- 修改配置文件，进行重启，将硬盘中的数据加载进内存，时间比较久。在这个过程中，Redis 不能提供服务。

- Redis 不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复。

- 主机宕机，宕机前有部分数据未能及时同步到从机，切换 IP 后还会引入数据不一致的问题，降低了系统的可用性。

- Redis 的主从复制采用全量复制，复制过程中主机会 fork 出一个子进程对内存做一份快照，并将子进程的内存快照保存为文件发送给从机，这一过程需要确保主机有足够多的空余内存。若快照文件较大，对集群的服务能力会产生较大的影响，而且复制过程是在从机新加入集群或者从机和主机网络断开重连时都会进行，也就是网络波动都会造成主机和从机间的一次全量的数据复制，这对实际的系统运营造成了不小的麻烦。

- Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

## Redis 适用场景

- 典型数据库（可持久化）

- 缓存

Redis 可以为每个键设置 TTL，到期后自动删除，即实现了缓存的功能，和典型的 KV 型同内存缓存 Memcached 不相上下。

- 队列

## 如何掌握 Redis 最小必须知识？

- **WHY/WHAT：明白 Redis 的优缺点，以及适用场景。**
- **HOW：理解 Redis 五种基本数据结构的工作原理及其正确使用，使用正确的数据结构处理特定的问题。**
- 使用编程语言驱动并使用 Redis。
- 掌握 Redis 主要的管理调试技巧

# redis-cli 通用命令

说明：`redis-cli` 命令不区分大小写。

- 连接服务器：`redis-cli -h 127.0.0.1 -p 6379`
- 测试连通情况：`redis-cli PING`，如果返回 `PONG` 则已经连通。
- 输出字符串：`ECHO hello`
- 停止 Redis Server: `redis-cli SHUTDOWN`
Redis 收到此命令后，会先断开所有客户端连接，然后根据配置执行持久化，最后退出。
- 选择数据库：`select n`，其中 `n` 是数据库编号
对 Redis 来说，一个数据库就是一个字典（根据 Redis 定义），对外都是从 0 开始的递增数字命名，默认 Redis 支持 16 个数据库，可以配置 `databases` 来修改。
- 获取当前数据库中符合指定规则的键名：`KEYS *`
- 判断一个键是否存在：`exists key`
- 删除指定 KEY：`del key1 key2 key3 ...`，返回删除的键的个数。
del 指令不支持通配符，但是可以这样曲线救国：
```
    redis-cli KEYS "user:*" | xargs redis-cli DEL
    redis-cli DEL `redis-cli KEYS "user:*"`
```
- 获得键值的数据类型：`TYPE key`
- 清空所有 KEY：`flushall`
- 清除所有值：`flushdb`

# Redis 五大数据结构

说明：

- 每个命令都相对应于一种特定的数据结构，这样的机制在 Redis 关键字集很小的情况下具有相当的可管理性。
- 一个命令，尤其是字符串相关的，只有在给定了明确的数据类型后，才会有实际意义。
- 使用各种数据结构时需要考虑是：如何利用散列数据结构去组织数据，是查询变得更高效。

## 字符串/string

字符串的值是标量。字符串适合存储对象和计数，也适合缓存数据。

```
    set users:caoxl "{name: caoxl, gender: sir}"
    
    strlen users:caoxl
    
    getrange users:caoxl 1 5
    
    append users:caoxl "balabala"
```

一些字符串命令是专门为一些类型或值的结构而设计的，比如整型数值。如下：

```
    incr stats:page:about
    incrby ratings:video:12333 5
```

## 散列表/hash

散列数据结构很像字符串，区别在于：hash 提供了一个额外的间接层：域（Field）。

```
    hset users:caoxl age 18
    hget users:caoxl age
    hmset users:caoxl age attr1 attr2 1024
    hmget users:caoxl age gender
    hgetall users:caoxl
    hkeys users:caoxl
    hdel users:caoxl age
```

## 列表/list

简而言之，列表就是数组。对于一个给定的关键字，列表让你可以存储和处理一组值，并对列表中的值进行 CRUD 和其他操作。

列表维护了值的顺序，提供了基于索引的高效操作。理论上列表可以存放任意个值。

```
    lpush newusers caoxl
    ltrim newusers 0 50
    lrange newusers 0 10
```

`LTRIM Key start stop`，删除指定范围外的值，时间复杂度 O(N)，N 是被删除的值的数量

这个例子中，由于先插入一个元素到列表开始（入栈）再进行列表调整，间接使得 `ltrim` 具有了常数性能，因为这种情况下 N 始终为 1。

## 集合/set

集合常被用来存储只能唯一存在的值，并提供了许多的基于集合的操作，例如交集、并集。集合没有对值进行操作，但是提供了高效的基于值的操作。

举例，使用集合实现朋友白名单：

```
    # 声明朋友集合
    sadd friends:cao cao_1 cao_2 cao_3 caoxl
    sadd friends:xl  xl_1  xl_2  xl_3  caoxl
    
    # 检查 X 是不是 Y 的朋友
    sismember friends:cao xl_1     # (integer) 0
    sismember friends:xl  cao_2    # (integer) 1 
    
    # 检查 N 个人是不是有共同的朋友
    sinter friends:cao friends:cj # 1) "caoxl"
    
    # 检查 N 个人是不是有共同的朋友 并把结果存在 friends:caoxl 集合中
    sinterstore friends:caoxl friends:cao friends:xl
```

有时候需要对值的属性进行标记和跟踪处理，但不能通过简单的复制操作完成，集合数据结构是解决此类问题的最好方法之一。

## 有序集合（Sorted Set）/zset

序集合类似集合，主要区分是标记（`score`）。标记提供了排序（`sorting`）和秩划分（`ranking`）

有序集合默认是根据 Score 对各个元素进行升序的排列。

```
    zadd friends:caoxl 60 caoxl_1 80 caoxl_2 100 caoxl_3
    zcount friends:caoxl 80 100    # (integer) 2
    zrank friends:caoxl caoxl_2    # 从低到高 获取 caoxl_2 在 friends_caoxl 中的 rank（秩）
    zrevrank friends:caoxl caoxl_2 # 从高到低 获取 caoxl_2 在 friends_caoxl 中的 rank（秩）
```

# 管理Redis

## 客户端验证

通过设置 `requirepass`，可以让 Redis 需要一个密码验证。验证也很简单，客户端只需要执行 `auth password` 就可以了。

`requirepass` 可以通过 `config set` 和修改 `redis.conf` 文件配置。

通过验证的客户端可以对 Redis 所有数据库执行任何命令，为了安全性，你可以设置关键字别名，比如把 config 命令设置为一个随机字符串：

```
    rename-command CONFIG 9336ebf25087d91c818ee6e9ec29f8c1 
    rename-command FLUSHALL 9dd4e461268c8034f5c8564e155c67a6
    rename-command FLUSHDB 415290769594460e2e485922904f345d
```

甚至，你可以将新名字设置为一个空字符串，从而禁掉某个命令。

## 数据库之间访问问题

Redis 不支持为每个数据库设置不同的访问密码，所以一个客户端要么可以访问全部数据要么全部都无权访问。

此外，多个数据库之间并非完全隔离的。比如 `FLUSHALL` 命令会清空该 Redis 实例中的所有数据。

因此，同一个 Redis 实例中不适合存储不同应用的数据，最好同一应用的不同环境也隔离开（保护生产数据），所有，**应该一个应用使用一个 Redis 实例**。

## 复制集/读写分离

所谓的复制集，指的是当我们向某个 Redis 实例（`master`）进行写入时，可以通过 `slaveof` 配置其他 Redis 实例（`slaves`）能够通过 master 保持更新。

为了数据安全性，也为了性能，应该将 Redis 复制到不同的服务器，实现异地冗余的同时，也提供了读写分离，因为读请求会被发送到 `slave` 实例。虽然数据可能出现滞后，但是大多数情况下都选择这种折中措施。

## 备份

备份 Redis 只需要把 Redis 的快照拷贝到任何地方即可。默认情况，Redis 的快照文件名是 `dump.rdb`。

为了减少 Master 实例的负载，在备份策略上通常把备份操作放到 slave，master 会停用快照功能以及单一附加文件（AOF）。


## 伸缩/集群

Redis 也能实现高可用和伸缩，以及集群间负载均衡，将命令负载转移或分散到多个 Slave 实例就可以了。不过，具体的效果需要根据实际情况针对性实现。

# 其他功能

## 字段超时

Redis 允许我们标记一个关键字的使用期限，从而实现缓存的作用。存活时间单位为秒或者时间戳。

```
    expire pages:about 30
    expireat pages:about 1504988562
    
    ttl pages:about # 查看某个字段还有多久过期
    setex pages:about '<h1>about us<h1> ...' # 设置字符串的同时设置过期时间 
    persist pages:about # 取消某个字段的超时设置
```

## 发布/订阅机制

Redis 提供的发布订阅机制可以在多个客户端之间实现通讯。

- 消费者订阅某个 channel

```
    subscribe music programming
```

- 生产者发布内容到某个 channel

```
    publish music 'jjlin new album'
    publish programming 'learn redis now'
```

## 慢日志

```
    config set slowlog-log-slower-than 0    # 记录所有命令
    slowlog get
    slowlog get 10
```

# 原理

## 什么是键值对？有什么含义？

键值对中键就是一切，所有的操作命令都是针对键的，不能根据值来操作。

值是关联到键的实际值，可以是任何东西。在大多数情况下，Redis 会把值看作一个字节序列，而不会关注他们实际上是什么。

## Redis 由哪些部分组成？

命令、关键字、值、数据结构以及与其一一绑定的操作方法。

## Redis 持久化原理？

默认情况，Redis 会根据已变更的关键字数量来进行判断，然后在磁盘里面创建数据库的快照（snapshot），如果 1000 个或者更多的关键字已经变更，Redis 会每隔 60 秒存储数据库，而如果9 个或者更少的关键字已变更，则每隔 15 分钟存储数据库。当然，这可以设置。

## 为什么 Redis 具有原子性？Redis 事务如何使用？

因为 Redis 是单线程的。每个 Redis 命令都具有原子性，也包括那些一次处理多项事情的命令。当一个命令在执行时，没有其他命令会运行。

常见一次处理多项事情的命令有：

- `incr`：先执行 get 命令再执行 set 命令。
- `getset`：先设置一个新的值，然后返回原始值。
- `setnx`：先测试关键字是否存在，只有当关键字不存在时才设置值。

此外，对于使用多个命令，Redis 支持事务功能。事务功能保证：

- 事务中的命令将会按顺序执行。
- 事务中的命令将会如单个原子操作般执行，没有其他客户端命令会在中途被执行。
- 事务中的命令要么全部执行，要么全不执行。

```
    multi    # 开始事务
    hincrby groups:1percent balance - 90000000
    hincrby groups:99percent balance 90000000
    exec    # 提交事务
    # discard    # 放弃执行
```

## 原子性的 Redis 还会遇到并发问题吗？

会，因为客户端是可以同时运行很多个的，当多个客户端操作同一个值的时候，就会出现并发问题。举例说明：

```
    def add():
        redis.multi()
        current = redis.get('sth')
        redis.set('sth', sth + 1)
        redis.exec()
    
    # 不安全的操作
    add()
    
    # 安全的操作（增加 watch）
    redis.watch('sth')
    add()
```

调用 `watch` 监控关键字之后，如果另一个客户端改变了 `sth` 的值，那么事务就会失败，我们可以在一个循环内运行这些代码，直到其能正常工作。

## 客户端与 Redis 实例之间的往返次数是不是越少越好？

不是，往往相反。SQL 的程序员通常会致力于让数据库的数据往返次数减至最小，这对于任何的数据库而言都是个好建议，但是考虑到我们是在处理比较简单的数据类型，有时候我们会频繁与 Redis 服务器交互，以实现我们的目的，通常情况下，在 Redis 高性能的前提下，这点性能损失可以忽略不计。

# FAQ

- 如何停止 `monitor`？

Redis 官方目前没有提供接口来专门处理这个问题，`ctrl + c` 无效，只有先断开连接，或者使用其他非官方的连接工具。

- `Docker redis container` 作为命令行客户端?

```
    docker run -it --link {REDIS_CONTAINER_NAME}:redis --rm redis redis-cli -h redis -p 6379
```

> 说明：生产环境尽量不要使用 monitor 命令，因为这是调试和开发工具。

- laravel 中使用 Redis 集群作为队列数据库？

hash tags, RTFM:<u>[https://redis.io/topics/cluster-spec#keys-hash-tags.](https://redis.io/topics/cluster-spec#keys-hash-tags)</u>

- Redis 和 Memcached 的区别？

**内存使用**：Memcahced 是全内存的缓存系统，Redis 既支持内存，也支持硬盘。此外，使用简单的key-value存储的话，Memcached的内存利用率更高，而如果Redis采用hash结构来做key-value存储，由于其组合式的压缩，其内存利用率会高于Memcached。

**性能上**：Redis 是单线程模型，而 Memcached 支持多线程，所以在多核服务器上以及大量数据存储时 Memcached 的性能理论上更高。在单核服务器上存储小量（100K内）数据，Redis 性能比 Memcached 高。

**功能上**：Redis 远比 Memcached 丰富，Redis 3.0+ 之后，Memcached 几乎所有功能都成了 Redis 的子集。

**集群**：Redis 原生支持集群，Memcached 需要借助第三方集群工具来实现。

**分布式**：两者都支持，Memcached 只能由客户端实现，Redis 更偏向在服务器构建。

**队列**：Redis 的类型键可用来实现队列，并且支持阻塞式读取，可以很容易地实现一个高性能的优先级队列。

**消息通讯**：Redis 支持发布/订阅模式，Memcached 无此机制。

# 参考

- [library/redis - Docker Hub](https://hub.docker.com/_/redis/)
- [阿里云Redis开发规范](https://m.aliyun.com/yunqi/articles/531067)

