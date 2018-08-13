---
title: Swoole 体验篇 (Socket服务器)
date: 2018-06-14 14:55:57
categories: Swoole
tags: Swoole
---

做的一个项目要用到 Socket 长链接，Google 了一番最终使用的是 [Swoole](https://wiki.swoole.com/)，这里简单介绍一下。

<!-- more -->


# 简介

## 使用场景

这里的需求是这样的，iOS 应用处于打开状态时，服务器能根据业务最新状态决定是否发送给该应用一些数据，
即：服务器在满足某些条件时，主动给客户端发送消息。

> 一般情况下指的只给在线用户推送消息.

## 短连接

在这种场景下，`App` 可以使用 `http` 接口主动轮询服务器（**短连接**），
然后根据服务器的返回状态做一些相应操作，但是这种做法有如下不足：

- 低效

假设很长时间都没有事件发生，即服务器很长时间都没有返回新的数据，但是 `App` 却总在向服务器发
相同的 `http` 请求（打开一个新的连接），那么这段无事发生的期间就是在做无用功。

- 浪费客户端和服务器双方资源

客户端发请求需要占用客户端资源，服务器响应需要占用服务器资源。

## 长连接

当然目前更高效的做法就是使用**长连接**。（`TCP`，非 `HTTP` ）

在每次客户端发出请求后，服务器检查上次返回的数据与此次请求时的数据之间是否有更新，
如果有更新则返回新数据并结束此次连接，否则服务器保持此次连接，直到有新数据时再返回相应

长链接比短链接高效的地方在于：减少了打开新连接个数和无用响应数。在保持长连接期间，
`C/S` 交互都使用的是同一个连接，这极大减少了资源浪费。

## 为什么选择 `Swoole` ?

处于对 `Swoole` 在圈内的名声和口碑的信心，我这里选择了 `Swoole` 来实现这个需求。

**`Swoole` 本质是一个标准的 `PHP` 扩展，但是又与普通的 `PHP` 扩展又有很大的区别：**

> 普通的扩展只是提供一个库函数。而`swoole`扩展在运行后会接管`PHP`的控制权，
进入事件循环。当IO事件发生后，`swoole`会自动回调指定的`PHP`函数。

`Swoole` 可以扮演的角色有很多：服务器、客户端、事件 API、异步 IO 接口、进程管理、内存区管理、高性能内存表等，
而这里，我只需要一个 `Socket` 服务器。

# 安装配置

> **请下载`releases`版本的`swoole`，直接从`github`主干上拉取最新代码可能会编译不过。**

```
    # 1.下载（https://github.com/swoole/swoole-src/releases）
    wget https://github.com/swoole/swoole-src/archive/1.8.10-stable.zip
    
    # 2.编译安装
    cd swoole-src-swoole-1.8.10-stable/
    phpize
    ./configure
    sudo make
    sudo make install
    
    # Or：`pecl install swoole`
    
    # 3. 配置 php.ini
    extension=swoole.so
    
    # 4. 检查
    php -m | grep swoole
```

说明：`swoole_server` 绝大部分功能只能用于 `php-cli` 环境，否则会抛出致命错误。

# 使用示例：服务器主动发送消息

这里只提供服务端的代码和返回的通信接口格式。
客户端和服务器通信只需要按照事先规定好的数据格式进行通信就行了。

```
    <?php
    
    class Server
    {
        private $db_conf = [
            'host' => '127.0.0.1',
            'port' => '3306',
            'name' => 'root',
            'pwd'  => '123456',
            'db'   => 'proj'
        ];
    
        private $secret = '10be8c337984d3a3c470f7cf5239c298';
    
        private $mysql  = null,
        private $serv;
    
        public function __construct()
        {
            $this->serv = new swoole_server("0.0.0.0", 9501);
            $this->serv->set(array(
                'worker_num' => 8,
                'daemonize'  => false,
            ));
    
            if ($this->serv && !$this->mysql)
                $this->mysqli_reconnect();
    
            if ($this->mysql && $this->mysql->set_charset('utf8')) {
                $this->serv->on('Start', array($this, 'onStart'));
                $this->serv->on('Connect', array($this, 'onConnect'));
                $this->serv->on('Receive', array($this, 'onReceive'));
                $this->serv->on('Close', array($this, 'onClose'));
    
                $this->serv->start();
            }
        }
    
        public function mysqli_reconnect()
        {
            $this->mysql = new mysqli(
                $this->db_conf['host'],
                $this->db_conf['name'],
                $this->db_conf['pwd'],
                $this->db_conf['db'],
                $this->db_conf['port']
            );
    
            return $this->mysql;
        }
    
        public function onStart( $serv )
        {
            // echo "Swoole socket Server Started. \n";
        }
    
        public function onConnect( $serv, $fd, $from_id)
        {
            // $serv->send( $fd, "Hello {$fd}!");
        }
    
        public function onReceive( swoole_server, $serv, $fd, $from_id, $data)
        {
            // echo "Get Message From Client {$fd}:{$data}\n";
    
            $response = [
                'nq_id'   => 'empty',
                'nq_data' => 'empty',
                'nq_ref'  => 'empty',
                'nq_type' => 'empty',
            ];
    
            if ($data) {
                $arr = json_decode($data, true);
    
                if ($arr && is_array($arr)
                        && (isset($arr['secret'])
                        && ($arr['secret'] == $this->secret))
                        && in_array($arr['option'], ['nothing', 'query', 'return'])
                        && isset($arr['uid']) && is_numeric($arr['uid'])) {
                    if ($arr['option'] == 'query') {
                        // 查询user_id 为 $arr['uid']的所有消息记录
                        $sql = 'select * from bm_notice_queues where nq_uid ='
                            .$arr['uid']
                            .'limit 1';
    
                        if ($this->mysql->errno == 2006) {
                            while(!$this->mysqli_reconnect()) {
                                $this->mysqli_reconnect();
                            }
                        }
    
                        $res = $this->mysql->query($sql);
    
                        if ($res && is_object($res) && $res->num_rows) {
                            if ($row = $res->fetch_assoc()) {
                                $response['nq_id']    = $row['nq_id'];
                                $response['nq_data']  = $row['nq_data'];
                                $response['nq_ref']   = $row['nq_ref'];
                                $response['nq_type']  = $row['nq_type'];
                            }
                        }
                    } elseif ($arr['option'] == 'return') {
                        if (($arr['status'] == 'success')
                            && is_numeric($arr['return'])
                            && $arr['return'] > 0) {
                            // 删除 id 为 $arr['return']的消息记录
                            $sql = 'delete from bm_notice_queues where nq_id = '.$arr['return'];
    
                            if ($this->mysql->errno == 2006) {
                                while(!$this->mysqli_reconnect()) {
                                    $this->mysqli_reconnect();
                                }
                            }
    
                            $this->mysql->query($sql);
                        }
                    }
                }
            }
    
            $serv->send($fd, json_encode($response));
        }
    
        public function onClose( $serv, $fd, $from_id) {
            // echo "Client {$fd} close connection\n";
        }
    }
    
    # 启动服务器
    $server = new Server();
```

# 可能用到的指令

- 定位到`php.ini`的绝对路径：

```
    php -i | grep php.ini
```

- 查看端口号是否被 监听

```
    netstat -an | grep <PORT>
```


# 参考

- [转载自:](http://blog.cjli.info/2016/07/16/Swoole-Socket-Intro/)
- [mac 安装 swoole扩展](http://www.cleey.com/blog/single/id/802.html)