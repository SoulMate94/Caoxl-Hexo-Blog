---
title: 心跳检测
date: 2018-06-13 15:43:58
categories: Internet
tags: [Internet, 心跳检测]
---

# 为什么需要心跳检测？ 

有些极端情况如客户端掉电、网络关闭、拔网线、路由故障等，这些极端情况都属于**连接断开**的情况，
然而这些情况如果没有应用层的心跳检测，服务端是无法快速感知的。
而**服务端定时向客户端发送心跳数据**可以解决这个问题。

<!-- more -->

# 心跳检测的原理是什么？

> 判断对方（设备，进程或其它网元）是否正常动行，一般采用定时发送简单的通讯包，
如果在指定时间段内未收到对方响应，则判断对方已经当掉。用于检测TCP的异常断开。

服务端向客户端发送心跳检测，客户端接收到心跳数据后，可以忽略不做任何处理，也可以回应心跳检测,

这就分为两种情况:

1. 当服务端不要求客户端必须回应心跳检测时，假如客户端遇到掉电等极端情况，
这时服务端向客户端发送的心跳数据在`TCP`层面就会发送超时，
遇到这种超时情况`TCP`会重试多次（**次数及间隔依赖操作系统的配置**），多次无果后会断开连接。
这种极端情况**从连接断开到服务端检测到可能要持续至少10分钟**。

2. 当服务端要求必须回应检测时，如果服务端在规定的时间内没有收到客户端的任何数据，
则立刻判定客户端已经断开，服务端就立即断开连接。

> 心跳检测可以是服务端主动，也可以是客户端主动，一般客户端来发会好点，对服务端压力没那么大

# "心跳"分为两种，第一种是客户端发起的心跳，第二种是服务端发起的心跳。

## 客户端发起的心跳

客户端每隔一段时间发送**策略消息**给`Socket`服务器，`Socket`服务器原路返回**策略消息**，
如果客户端在设定时间段内没有收到`Socket`服务器的返回消息，经重试机制后，判定`Socket`服务器已`Down`，关闭连接。

## 服务端发起的心跳

服务端实时记录每条`Socket`的`IO`操作时间，每隔一段时间获取所有`Socket`列表的快照，扫描每条`Socket`，
如果该`Socket`的`IO`操作时间距当前时间已超出设定值，则判定客户端`Down`，关闭连接。

# 基于`Workerman`的`PHP`心跳

```
    <?php
    
    require_once __DIR__.'/Workerman/Autoloader.php';
    
    use Workerman\Worker;
    use Workerman\Lib\Timer;
    
    define('HEARTBEAT_TIME', 25);   //心跳间隔25秒
    
    $worker = new Worker('text://0.0.0.0:1234');
    
    $worker->onMessage = function ($connection, $msg) {
        // 给connection临时设置一个lastMessageTime属性，用来记录上次收到消息的时间
        $connection->lastMessageTime = time();
        //
    };
    
    // 进程启动后设置一个每秒运行一次的定时器
    $worker->onWorkerStart = function($worker) {
        Timer::add(1, function () use ($worker) {
            $time_now = time();
            foreach ($worker->connections as $connection) {
                // 有可能该connection还没收到过消息，则lastMessageTime设置为当前时间
                if (empty($connection->lastMessageTime)) {
                    $connection->lastMessageTime = $time_now;
                    continue;
                }
    
                // 上次通讯时间间隔大于心跳间隔，则认为客户端已经下线，关闭连接
                if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                    $connection->close();
                }
            }
        });
    };
    
    Worker::runAll();
```

# `Swoole`做心跳检测

事实上，`Swoole`已经内置了心跳检测功能，能自动`close`掉长时间没有数据来往的连接。
而开启心跳检测功能，只需要设置`heartbeat_check_interval`和`heartbeat_idle_time`即可。如下：

```
    $this->serv->set(
        array(
            'headerbeat_check_interval' => 60,
            'headerbeat_idle_time'      => 600,
        )
    );
```

其中`heartbeat_idle_time`的默认值是`heartbeat_check_interval`的两倍。 
在设置这两个选项后，`swoole`会在内部启动一个线程，每隔`heartbeat_check_interval`秒后遍历一次全部连接，检查最近一次发送数据的时间和当前时间的差，
如果这个差值大于`heartbeat_idle_time`，则会强制关闭这个连接，并通过回调`onClose`通知`Server`进程
结合之前的`Timer`功能，如果我们想维持连接，就设置一个略小于如果这个差值大于`heartbeat_idle_time`的定时器，
在定时器内向所有连接发送一个心跳包。如果收到心跳回应，则判断连接正常，如果没有收到，则关闭这个连接或者再次尝试发送。


> 互联网推送原理: 长连接＋心跳机制

# 参考

- [workerman心跳检测原理](https://blog.csdn.net/nuli888/article/details/51946553)
- [Timer定时器、心跳检测及Task进阶实例：mysql连接池](https://github.com/LinkedDestiny/swoole-doc/blob/master/03.Timer%E5%AE%9A%E6%97%B6%E5%99%A8%E3%80%81%E5%BF%83%E8%B7%B3%E6%A3%80%E6%B5%8B%E5%8F%8ATask%E8%BF%9B%E9%98%B6%E5%AE%9E%E4%BE%8B%EF%BC%9Amysql%E8%BF%9E%E6%8E%A5%E6%B1%A0.md#2%E5%BF%83%E8%B7%B3%E6%A3%80%E6%B5%8B)