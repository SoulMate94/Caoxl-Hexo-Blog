---
title: Redis 命令
date: 2018-03-28 14:55:33
categories: Redis
tags: Redis
---

记录一些常用的Redis命令和例子。

<!--more-->


# Key (键)

## DEL  删除给定的一个或多个 key 。

```php
    redis> SET name huangz
    OK

    redis> DEL name
    (integer) 1
```

## DUMP 序列化给定 key ，并返回被序列化的值

使用 RESTORE 命令可以将这个值反序列化为 Redis 键。

```php
    redis> SET greeting "hello, dumping world!"
    OK

    redis> DUMP greeting
    "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"

    redis> DUMP not-exists-key
    (nil)
```

## EXISTS 检查给定 key 是否存在。

```php
    redis> SET db "redis"
    OK

    redis> EXISTS db
    (integer) 1

    redis> DEL db
    (integer) 1

    redis> EXISTS db
    (integer) 0
```

## EXPIRE  为给定 key 设置生存时间

当 key 过期时(生存时间为 0 )，它会被自动删除。

```php
    redis> SET cache_page "www.google.com"
    OK

    redis> EXPIRE cache_page 30  # 设置过期时间为 30 秒
    (integer) 1

    redis> TTL cache_page    # 查看剩余生存时间
    (integer) 23

    redis> EXPIRE cache_page 30000   # 更新过期时间
    (integer) 1

    redis> TTL cache_page
    (integer) 29996
```

## EXPIREAT  用于为 key 设置生存时间

```php
    redis> SET cache www.google.com
    OK

    redis> EXPIREAT cache 1355292000     # 这个 key 将在 2012.12.12 过期
    (integer) 1

    redis> TTL cache
    (integer) 45081860
```

不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)

## KEYS 查找所有符合给定模式 pattern 的 key 。

> KEYS * 匹配数据库中所有 key 。
KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。
KEYS h * llo 匹配 hllo 和 heeeeello 等。
KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo 。

```php
    redis> MSET one 1 two 2 three 3 four 4  # 一次设置 4 个 key
    OK

    redis> KEYS *o*
    1) "four"
    2) "two"
    3) "one"

    redis> KEYS t??
    1) "two"

    redis> KEYS t[w]*
    1) "two"

    redis> KEYS *  # 匹配数据库内所有 key
    1) "four"
    2) "three"
    3) "two"
```


## MIGRATE 将 key 原子性地从当前实例传送到目标实例的指定数据库上

一旦传送成功， key 保证会出现在目标实例上，而当前实例上的 key 会被删除。

这个命令是一个原子操作，它在执行的时候会阻塞进行迁移的两个实例，直到以下任意结果发生：`迁移成功`，`迁移失败`，`等待超时`。


```php
    $ ./redis-cli

    redis 127.0.0.1:6379> flushdb
    OK

    redis 127.0.0.1:6379> SET greeting "Hello from 6379 instance"
    OK

    redis 127.0.0.1:6379> MIGRATE 127.0.0.1 7777 greeting 0 1000
    OK

    redis 127.0.0.1:6379> EXISTS greeting                           # 迁移成功后 key 被删除
    (integer) 0
```


## MOVE  将当前数据库的 key 移动到给定的数据库 db 当中。

```php
    # key 存在于当前数据库

    redis> SELECT 0                             # redis默认使用数据库 0，为了清晰起见，这里再显式指定一次。
    OK

    redis> SET song "secret base - Zone"
    OK

    redis> MOVE song 1                          # 将 song 移动到数据库 1
    (integer) 1

    redis> EXISTS song                          # song 已经被移走
    (integer) 0

    redis> SELECT 1                             # 使用数据库 1
    OK

    redis:1> EXISTS song                        # 证实 song 被移到了数据库 1 (注意命令提示符变成了"redis:1"，表明正在使用数据库 1)
    (integer) 1
```


## OBJECT  命令允许从内部察看给定 key 的 Redis 对象。

```php
    redis> SET game "COD"           # 设置一个字符串
    OK

    redis> OBJECT REFCOUNT game     # 只有一个引用
    (integer) 1

    redis> OBJECT IDLETIME game     # 等待一阵。。。然后查看空闲时间
    (integer) 90

    redis> GET game                 # 提取game， 让它处于活跃(active)状态
    "COD"

    redis> OBJECT IDLETIME game     # 不再处于空闲状态
    (integer) 0

    redis> OBJECT ENCODING game     # 字符串的编码方式
    "raw"

    redis> SET big-number 23102930128301091820391092019203810281029831092  # 非常长的数字会被编码为字符串
    OK

    redis> OBJECT ENCODING big-number
    "raw"

    redis> SET small-number 12345  # 而短的数字则会被编码为整数
    OK

    redis> OBJECT ENCODING small-number
    "int"
```


## PERSIST 移除给定 key 的生存时间

将这个 key 从『易失的』(带生存时间 key )转换成『持久的』(一个不带生存时间、永不过期的 key )

```php
    redis> SET mykey "Hello"
    OK

    redis> EXPIRE mykey 10  # 为 key 设置生存时间
    (integer) 1

    redis> TTL mykey
    (integer) 10

    redis> PERSIST mykey    # 移除 key 的生存时间
    (integer) 1

    redis> TTL mykey
    (integer) -1
```


## PEXPIRE  以毫秒为单位设置 key 的生存时间，而不像 EXPIRE 命令那样，以秒为单位。

```php
    redis> SET mykey "Hello"
    OK

    redis> PEXPIRE mykey 1500
    (integer) 1

    redis> TTL mykey    # TTL 的返回值以秒为单位
    (integer) 2

    redis> PTTL mykey   # PTTL 可以给出准确的毫秒数
    (integer) 1499
```

## PEXPIREAT  以毫秒为单位设置 key 的过期 unix 时间戳，而不是像 EXPIREAT 那样，以秒为单位。

这个命令和 EXPIREAT 命令类似

```php
    redis> SET mykey "Hello"
    OK

    redis> PEXPIREAT mykey 1555555555005
    (integer) 1

    redis> TTL mykey           # TTL 返回秒
    (integer) 223157079

    redis> PTTL mykey          # PTTL 返回毫秒
    (integer) 223157079318
```


## PTTL 以毫秒为单位返回 key 的剩余生存时间，而不是像 TTL 命令那样，以秒为单位

```php
    # 不存在的 key

    redis> FLUSHDB
    OK

    redis> PTTL key
    (integer) -2


    # key 存在，但没有设置剩余生存时间

    redis> SET key value
    OK

    redis> PTTL key
    (integer) -1


    # 有剩余生存时间的 key

    redis> PEXPIRE key 10086
    (integer) 1

    redis> PTTL key
    (integer) 6179
```

## RANDOMKEY  从当前数据库中随机返回(不删除)一个 key 。

```php
    # 数据库不为空

    redis> MSET fruit "apple" drink "beer" food "cookies"   # 设置多个 key
    OK

    redis> RANDOMKEY
    "fruit"

    redis> RANDOMKEY
    "food"

    redis> KEYS *    # 查看数据库内所有key，证明 RANDOMKEY 并不删除 key
    1) "food"
    2) "drink"
    3) "fruit"


    # 数据库为空

    redis> FLUSHDB  # 删除当前数据库所有 key
    OK

    redis> RANDOMKEY
    (nil)
```

## RENAME 将 key 改名为 newkey 。

当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。

当 newkey 已经存在时， RENAME 命令将覆盖旧值。

```php
    # key 存在且 newkey 不存在

    redis> SET message "hello world"
    OK

    redis> RENAME message greeting
    OK

    redis> EXISTS message               # message 不复存在
    (integer) 0

    redis> EXISTS greeting              # greeting 取而代之
    (integer) 1


    # 当 key 不存在时，返回错误

    redis> RENAME fake_key never_exists
    (error) ERR no such key


    # newkey 已存在时， RENAME 会覆盖旧 newkey

    redis> SET pc "lenovo"
    OK

    redis> SET personal_computer "dell"
    OK

    redis> RENAME pc personal_computer
    OK

    redis> GET pc
    (nil)

    redis:1> GET personal_computer      # 原来的值 dell 被覆盖了
    "lenovo"
```


## RENAMENX  当且仅当 newkey 不存在时，将 key 改名为 newkey 。

当 key 不存在时，返回一个错误。

```php
    # newkey 不存在，改名成功

    redis> SET player "MPlyaer"
    OK

    redis> EXISTS best_player
    (integer) 0

    redis> RENAMENX player best_player
    (integer) 1


    # newkey存在时，失败

    redis> SET animal "bear"
    OK

    redis> SET favorite_animal "butterfly"
    OK

    redis> RENAMENX animal favorite_animal
    (integer) 0

    redis> get animal
    "bear"

    redis> get favorite_animal
    "butterfly"
```


## RESTOR  反序列化给定的序列化值，并将它和给定的 key 关联。

```php
    # 创建一个键，作为 DUMP 命令的输入

    redis> SET greeting "hello, dumping world!"
    OK

    redis> DUMP greeting
    "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"

    # 将序列化数据 RESTORE 到另一个键上面

    redis> RESTORE greeting-again 0 "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"
    OK

    redis> GET greeting-again
    "hello, dumping world!"

    # 在没有给定 REPLACE 选项的情况下，再次尝试反序列化到同一个键，失败

    redis> RESTORE greeting-again 0 "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"
    (error) ERR Target key name is busy.

    # 给定 REPLACE 选项，对同一个键进行反序列化成功

    redis> RESTORE greeting-again 0 "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde" REPLACE
    OK

    # 尝试使用无效的值进行反序列化，出错

    redis> RESTORE fake-message 0 "hello moto moto blah blah"
    (error) ERR DUMP payload version or checksum are wrong
```

## SORT  返回或保存给定列表、集合、有序集合 key 中经过排序的元素。

```php
    # 开销金额列表

    redis> LPUSH today_cost 30 1.5 10 8
    (integer) 4

    # 排序

    redis> SORT today_cost
    1) "1.5"
    2) "8"
    3) "10"
    4) "30"

    # 逆序排序

    redis 127.0.0.1:6379> SORT today_cost DESC
    1) "30"
    2) "10"
    3) "8"
    4) "1.5"
```

## TTL  以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。

```php
    # 不存在的 key

    redis> FLUSHDB
    OK

    redis> TTL key
    (integer) -2


    # key 存在，但没有设置剩余生存时间

    redis> SET key value
    OK

    redis> TTL key
    (integer) -1


    # 有剩余生存时间的 key

    redis> EXPIRE key 10086
    (integer) 1

    redis> TTL key
    (integer) 10084
```

## TYPE  返回 key 所储存的值的类型。

```php
    # 字符串

    redis> SET weather "sunny"
    OK

    redis> TYPE weather
    string


    # 列表

    redis> LPUSH book_list "programming in scala"
    (integer) 1

    redis> TYPE book_list
    list


    # 集合

    redis> SADD pat "dog"
    (integer) 1

    redis> TYPE pat
    set
```

## SCAN  命令及其相关的 SSCAN 命令、 HSCAN 命令和 ZSCAN 命令都用于增量地迭代

- SCAN 命令用于迭代当前数据库中的数据库键。
- SSCAN 命令用于迭代集合键中的元素。
- HSCAN 命令用于迭代哈希键中的键值对。
- ZSCAN 命令用于迭代有序集合中的元素（包括元素成员和元素分值）。



# String (字符串)

## APPEND  将 value 追加到 key 原来的值的末尾

如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。

如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样。

```php
    # 对不存在的 key 执行 APPEND

    redis> EXISTS myphone               # 确保 myphone 不存在
    (integer) 0

    redis> APPEND myphone "nokia"       # 对不存在的 key 进行 APPEND ，等同于 SET myphone "nokia"
    (integer) 5                         # 字符长度


    # 对已存在的字符串进行 APPEND

    redis> APPEND myphone " - 1110"     # 长度从 5 个字符增加到 12 个字符
    (integer) 12

    redis> GET myphone
    "nokia - 1110"
```

## BITCOUNT  计算给定字符串中，被设置为 1 的比特位的数量。

```php
    redis> BITCOUNT bits
    (integer) 0

    redis> SETBIT bits 0 1          # 0001
    (integer) 0

    redis> BITCOUNT bits
    (integer) 1

    redis> SETBIT bits 3 1          # 1001
    (integer) 0

    redis> BITCOUNT bits
    (integer) 2
```

## BITOP  对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。

```php
    redis> SETBIT bits-1 0 1        # bits-1 = 1001
    (integer) 0

    redis> SETBIT bits-1 3 1
    (integer) 0

    redis> SETBIT bits-2 0 1        # bits-2 = 1011
    (integer) 0

    redis> SETBIT bits-2 1 1
    (integer) 0

    redis> SETBIT bits-2 3 1
    (integer) 0

    redis> BITOP AND and-result bits-1 bits-2
    (integer) 1

    redis> GETBIT and-result 0      # and-result = 1001
    (integer) 1

    redis> GETBIT and-result 1
    (integer) 0

    redis> GETBIT and-result 2
    (integer) 0

    redis> GETBIT and-result 3
    (integer) 1
```


## DECR  将 key 中储存的数字值减一。

```php
    # 对存在的数字值 key 进行 DECR

    redis> SET failure_times 10
    OK

    redis> DECR failure_times
    (integer) 9


    # 对不存在的 key 值进行 DECR

    redis> EXISTS count
    (integer) 0

    redis> DECR count
    (integer) -1


    # 对存在但不是数值的 key 进行 DECR

    redis> SET company YOUR_CODE_SUCKS.LLC
    OK

    redis> DECR company
    (error) ERR value is not an integer or out of range
```

## DECRBY  将 key 所储存的值减去减量 decrement 。

```php
    # 对已存在的 key 进行 DECRBY

    redis> SET count 100
    OK

    redis> DECRBY count 20
    (integer) 80


    # 对不存在的 key 进行DECRBY

    redis> EXISTS pages
    (integer) 0

    redis> DECRBY pages 10
    (integer) -10
```


## GET  返回 key 所关联的字符串值。


```php
    # 对不存在的 key 或字符串类型 key 进行 GET

    redis> GET db
    (nil)

    redis> SET db redis
    OK

    redis> GET db
    "redis"


    # 对不是字符串类型的 key 进行 GET

    redis> DEL db
    (integer) 1

    redis> LPUSH db redis mongodb mysql
    (integer) 3

    redis> GET db
    (error) ERR Operation against a key holding the wrong kind of value
```


## GETBIT  对 key 所储存的字符串值，获取指定偏移量上的位(bit)。

```php
    # 对不存在的 key 或者不存在的 offset 进行 GETBIT， 返回 0

    redis> EXISTS bit
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 0


    # 对已存在的 offset 进行 GETBIT

    redis> SETBIT bit 10086 1
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 1
```


## GETRANGE  返回 key 中字符串值的子字符串

字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)。

```php
    redis> SET greeting "hello, my friend"
    OK

    redis> GETRANGE greeting 0 4          # 返回索引0-4的字符，包括4。
    "hello"

    redis> GETRANGE greeting -1 -5        # 不支持回绕操作
    ""

    redis> GETRANGE greeting -3 -1        # 负数索引
    "end"

    redis> GETRANGE greeting 0 -1         # 从第一个到最后一个
    "hello, my friend"

    redis> GETRANGE greeting 0 1008611    # 值域范围不超过实际字符串，超过部分自动被符略
    "hello, my friend"
```

## GETSET  将给定 key 的值设为 value ，并返回 key 的旧值(old value)。

当 key 存在但不是字符串类型时，返回一个错误。

```php
    redis> GETSET db mongodb    # 没有旧值，返回 nil
    (nil)

    redis> GET db
    "mongodb"

    redis> GETSET db redis      # 返回旧值 mongodb
    "mongodb"

    redis> GET db
    "redis"
```

## INCR  将 key 中储存的数字值增一。

```php
    redis> SET page_view 20
    OK

    redis> INCR page_view
    (integer) 21

    redis> GET page_view    # 数字值在 Redis 中以字符串的形式保存
    "21"
```


## INCRBY  将 key 所储存的值加上增量 increment 。

```php
    # key 存在且是数字值

    redis> SET rank 50
    OK

    redis> INCRBY rank 20
    (integer) 70

    redis> GET rank
    "70"


    # key 不存在时

    redis> EXISTS counter
    (integer) 0

    redis> INCRBY counter 30
    (integer) 30

    redis> GET counter
    "30"


    # key 不是数字值时

    redis> SET book "long long ago..."
    OK

    redis> INCRBY book 200
```


## INCRBYFLOAT  为 key 中所储存的值加上浮点数增量 increment 。

```php
    # 值和增量都不是指数符号

    redis> SET mykey 10.50
    OK

    redis> INCRBYFLOAT mykey 0.1
    "10.6"


    # 值和增量都是指数符号

    redis> SET mykey 314e-2
    OK

    redis> GET mykey                # 用 SET 设置的值可以是指数符号
    "314e-2"

    redis> INCRBYFLOAT mykey 0      # 但执行 INCRBYFLOAT 之后格式会被改成非指数符号
    "3.14"


    # 可以对整数类型执行

    redis> SET mykey 3
    OK

    redis> INCRBYFLOAT mykey 1.1
    "4.1"


    # 后跟的 0 会被移除

    redis> SET mykey 3.0
    OK

    redis> GET mykey                                    # SET 设置的值小数部分可以是 0
    "3.0"

    redis> INCRBYFLOAT mykey 1.000000000000000000000    # 但 INCRBYFLOAT 会将无用的 0 忽略掉，有需要的话，将浮点变为整数
    "4"

    redis> GET mykey
    "4"
```

## MGET  返回所有(一个或多个)给定 key 的值。

```php
    redis> SET redis redis.com
    OK

    redis> SET mongodb mongodb.org
    OK

    redis> MGET redis mongodb
    1) "redis.com"
    2) "mongodb.org"

    redis> MGET redis mongodb mysql     # 不存在的 mysql 返回 nil
    1) "redis.com"
    2) "mongodb.org"
    3) (nil)
```

## MSET  同时设置一个或多个 key-value 对

```php
    redis> MSET date "2012.3.30" time "11:00 a.m." weather "sunny"
    OK

    redis> MGET date time weather
    1) "2012.3.30"
    2) "11:00 a.m."
    3) "sunny"


    # MSET 覆盖旧值例子

    redis> SET google "google.hk"
    OK

    redis> MSET google "google.com"
    OK

    redis> GET google
"google.com"
```

## MSETNX  同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。

```php
    # 对不存在的 key 进行 MSETNX

    redis> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
    (integer) 1

    redis> MGET rmdbs nosql key-value-store
    1) "MySQL"
    2) "MongoDB"
    3) "redis"


    # MSET 的给定 key 当中有已存在的 key

    redis> MSETNX rmdbs "Sqlite" language "python"  # rmdbs 键已经存在，操作失败
    (integer) 0

    redis> EXISTS language                          # 因为 MSET 是原子性操作，language 没有被设置
    (integer) 0

    redis> GET rmdbs                                # rmdbs 也没有被修改
    "MySQL"
```


## SET  将字符串值 value 关联到 key 。

```php
    # 对不存在的键进行设置

    redis 127.0.0.1:6379> SET key "value"
    OK

    redis 127.0.0.1:6379> GET key
    "value"


    # 对已存在的键进行设置

    redis 127.0.0.1:6379> SET key "new-value"
    OK

    redis 127.0.0.1:6379> GET key
    "new-value"

```

# Hash (哈希表)


# List (列表)
 

# Set (集合)


# SortedSet (有序集合)

# HyperLogLog

# GEO (地理位置)

# Pub/Sub (发布、订阅)

# Transaction (事务)

# Script (脚本)

# Connection (连接)

# Server (服务器)

# 参考

- [Redis 命令参考](http://redisdoc.com/)