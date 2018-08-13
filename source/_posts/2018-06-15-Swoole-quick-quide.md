---
title: Swoole 快速起步
date: 2018-06-15 15:02:51
categories: Swoole
tags: Swoole
---

`Swoole`的绝大部分功能只能用于`cli`命令行环境，请首先准备好`Linux Shell`环境。
可使用`vim`、`emacs`、`phpstorm`或其他编辑器编写代码，并在命令行中通过下列指令执行程序。

<!-- more -->

> 以下是根据文档,进行实操,手敲文档.

# 创建`TCP`服务器

```TCP
    // 创建Server对象, 监听127.0.0.1:9501端口
    $serv = new swoole_server("127.0.0.1", 9501);
    
    // 监听连接进入事件
    $serv->on('connect', function ($serv, $fd) {
        echo "Client: Connection.\n";
    });
    
    // 监听数据接收事件
    $serv->on('receive', function ($serv, $fd, $from_id, $data) {
        $serv->send($fd, "Server: ".$data);
    });
    
    // 监听连接关闭事件
    $serv->on('close', function ($serv, $fd) {
        echo "Client: Close.\n";
    });
    
    // 启动服务器
    $serv->start();
```

> 这里就创建了一个TCP服务器，监听本机9501端口。它的逻辑很简单，
当客户端`Socket`通过网络发送一个 `hello` 字符串时，服务器会回复一个 `Server: hello` 字符串。

```
    [root@izj6c6djex81rijczh0t8yz ~]# telnet 127.0.0.1 9501
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.
    hello   
    Server: hello
```

**swoole_server是异步服务器**，所以是通过监听事件的方式来编写程序的。当对应的事件发生时底层会主动回调指定的`PHP`函数。
如当有新的`TCP`连接进入时会执行`onConnect`事件回调，当某个连接向服务器发送数据时会回调`onReceive`函数

- 服务器可以同时被成千上万个客户端连接，`$fd`就是客户端连接的唯一标识符
- 调用 `$server->send()` 方法向客户端连接发送数据，参数就是`$fd`客户端标识符
- 调用 `$server->close()` 方法可以强制关闭某个客户端连接
- 客户端可能会主动断开连接，此时会触发`onClose`事件回调

## 测试

```
    telnet 127.0.0.1 9501
    hello
    Server: hello
```

# 创建`UDP`服务器

```
    // 创建server对象, 监听127.0.0.1:9502端口, 类型为SWOOLE_SOCK_UDP
    $serv = new swoole_server("127.0.0.1", 9502, SWOOLE_PROCESS, SWOOLE_SOCK_UDP);
    
    // 监听数据接收事件
    $serv->on('Packet', function ($serv, $data, $clientInfo) {
        $serv->sendto($clientInfo['address'], $clientInfo['port'], "Server".$data);
        var_dump($clientInfo);
    });
    
    // 启动服务器
    $serv->start();
```

> `UDP`服务器与`TCP`服务器不同，`UDP`没有连接的概念。启动`Server`后，客户端无需`Connect`，
直接可以向`Server`监听的`9502`端口发送数据包。对应的事件为`onPacket`。

- `$clientInfo`是客户端的相关信息，是一个数组，有客户端的IP和端口等内容
- 调用 `$server->sendto` 方法向客户端发送数据

## 测试

```
    netcat -u 127.0.0.1 9502
    hello
    Server: hello
```

# 创建`WEB`服务器

```
    $http = new swoole_http_server("0.0.0.0", 9501);
    
    $http->on('request', function ($request, $response) {
        var_dump($request->get, $request->post);
        $response->header("Content-type", "text/html;charset=utf-8");
        $response->end("<h1>Hello Swoole.#</h1>".rand(1000, 9999)."</h1>");
    });
    
    $http->start();
```

`Http`服务器只需要关注请求响应即可，所以只需要监听一个`onRequest`事件。当有新的`Http`请求进入就会触发此事件

事件回调函数有`2`个参数: 一个是`$request`对象，包含了请求的相关信息，如`GET`/`POST`请求的数据。

另外一个是`response`对象，对`request`的响应可以通过操作`response`对象来完成。
`$response->end()`方法表示输出一段HTML内容，并结束此请求。

- `0.0.0.0` 表示监听所有IP地址，一台服务器可能同时有多个IP，
如`127.0.0.1`本地回环IP、`192.168.1.100`局域网IP、`210.127.20.2` 外网IP，这里也可以单独指定监听一个IP
- `9501` 监听的端口，如果被占用程序会抛出致命错误，中断执行。

## 测试

- 可以打开浏览器，访问`http://127.0.0.1:9501`查看程序的结果。
- `Linux`下: `curl http://127.0.0.1:9501`
- 也可以使用`apache ab`工具对服务器进行压力测试


# 创建`WebSocket`服务器

```
   // 创建WebSocket服务器对象, 监听0.0.0.0:9502端口
   $ws = new swoole_websocket_server("0.0.0.0", 9502);
   
   // 监听WebSocket连接打开事件
   $ws->on('open', function ($ws, $request) {
       var_dump($request->fd, $request->get, $request->server);
       $ws->push($request->fd, "hello, welcome\n");
   });
   
   // 监听WebSocket消息事件
   $ws->on('message', function ($ws, $frame) {
       echo "Message: {$frame->data}\n";
       $ws->push($frame->fd, "server: {$frame->data}");
   });
   
   // 监听WebSocket连接关闭事件
   $ws->on('close', function ($ws, $fd) {
       echo "client-{$fd} is closed\n";
   });
   
   $ws->start(); 
```

> `WebSocket`服务器是建立在Http服务器之上的长连接服务器，客户端首先会发送一个`Http`的请求与服务器进行握手。
握手成功后会触发`onOpen`事件，表示连接已就绪，`onOpen`函数中可以得到`$request`对象，
包含了`Http`握手的相关信息，如`GET参数`、`Cookie`、`Http头信息`等。

建立连接后客户端与服务器端就可以双向通信了。

- 客户端向服务器端发送信息时，服务器端触发`onMessage`事件回调
- 服务器端可以调用`$server->push()`向某个客户端（使用`$fd`标识符）发送消息
- 服务器端可以设置`onHandShake`事件回调来手工处理WebSocket握手

## 测试

```
    // JS
    var wsServer  = 'ws://127.0.0.1:9502';
    var websocket = new WebSocket(wsServer);

    websocket.onopen = function (evt) {
        console.log("Connected to WebSocket server.");
    }

    websocket.onclose = function (evt) {
        console.log("Disconnected");
    }

    websocket.onmessage = function (evt) {
        console.log('Retrieved data from server:' + evt.data);
    }

    websocket.onerror = function (evt, e) {
        console.log('Error occured: ' + evt.data);
    }
```

- 不能直接使用`swoole_client`与`websocket`服务器通信，`swoole_client`是`TCP`客户端
- 必须实现`WebSocket`协议才能和`WebSocket`服务器通信，可以使用`swoole/framework`提供的[PHP WebSocket客户端](https://github.com/swoole/framework/blob/master/libs/Swoole/Client/WebSocket.php)

> `WebSocket`服务器除了提供`WebSocket`功能之外，
实际上也可以处理`Http`长连接。只需要增加`onRequest`事件监听即可实现`Comet`方案`Http长轮询`

# 设置定时器

`swoole`提供了类似`JavaScript`的`setInterval`/`setTimeout`异步高精度定时器，粒度为毫秒级。使用也非常简单。

```php  
    // 每隔2000ms触发一次
    swoole_timer_tick(2000, function ($timer_id) {
        echo "tick-2000ms\n";
    });
    
    // 3000ms后执行此函数
    swoole_timer_after(3000, function () {
        echo "after 3000ms.\n";
    });
```

- `swoole_timer_tick`函数就相当于`setInterval`，是持续触发的
- `swoole_timer_after`函数相当于`setTimeout`，仅在约定的时间触发一次
- `swoole_timer_tick`和`swoole_timer_after`函数会返回一个整数，表示定时器的`ID`
- 可以使用 `swoole_timer_clear` 清除此定时器，参数为定时器ID

## 实例

```
    public function onWorkerStart($serv, $worker_id)
    {
        $this->redis = new Redis;
        $this->redis->connect('127.0.0.1', 6379);
        $this->redis->auth('123456');
        $this->redis->flushAll();

        if ($worker_id === 0) {
            $serv->tick(1000, function ($id) use ($serv) {
                foreach ($serv->connections as $fd) {
                    $serv->push($fd, json_encode([
                        'err' => 1,
                    ]));
                }
            });
        }
    }
```

# 执行异步任务

> 在Server程序中如果需要执行很耗时的操作，比如一个聊天服务器发送广播，Web服务器中发送邮件。
如果直接去执行这些函数就会阻塞当前进程，导致服务器响应变慢。

`Swoole`提供了异步任务处理的功能，可以**投递一个异步任务到TaskWorker进程池中执行**，不影响当前请求的处理速度。

基于第一个`TCP`服务器，只需要增加`onTask`和`onFinish`2个事件回调函数即可。
另外需要设置`task`进程数量，可以根据任务的耗时和任务量配置适量的task进程。

```
    $serv = new swoole_server("127.0.0.1", 9501);
    
    // 设置异步任务的工作进程数量
    $serv->set(array('task_worker_num' => 4));
    
    $serv->on('receive', function($serv, $fd, $from_id, $data) {
        // 投递异步任务
        $task_id = $serv->task($data);
        echo "Dispath AsynTask: id=$task_id\n";
    });
    
    // 处理异步任务
    $serv->on('task', function($serv, $task_id, $from_id, $data) {
        echo "New AsynTask[id=$task_id]".PHP_EOL;
    
        // 返回任务执行的结果
        $serv->finish("$data-> OK");
    });
    
    // 处理异步任务的结果
    $serv->on('finish', function ($serv, $task_id, $data) {
        echo "AsynTask[$task_id] Finish: $data".PHP_EOL;
    });
    
    $serv->start();
```

调用`$serv->task()`后，程序立即返回，继续向下执行代码。
`onTask`回调函数`Task`进程池内被异步执行。执行完成后调用`$serv->finish()`返回结果。

> `finish`操作是可选的，也可以不返回任何结果

# 创建同步`TCP`客户端

```
    $client = new swoole_client(SWOOLE_SOCK_TCP);
    
    // 连接到服务器
    if (!$client->connect('127.0.0.1', 9501, 0.5)) {
        die("Connect failed.");
    }
    
    // 向服务器发送数据
    if (!$client->send("hello world")) {
        die("Send failed.");
    }
    
    // 从服务器接收数据
    $data = $client->recv();
    if (!$data) {
        die("Recv failed.");
    }
    
    echo $data;
    
    // 关闭连接
    $client->close();
```

创建一个`TCP`的同步客户端，此客户端可以用于连接到我们第一个示例的`TCP`服务器。
向服务器端发送一个`hello world`字符串，服务器会返回一个 `Server: hello world`字符串。

个客户端是同步阻塞的，`connect/send/recv` 会等待IO完成后再返回。**同步阻塞操作并不消耗CPU资源**，
IO操作未完成当前进程会自动转入`sleep`模式，**当IO完成后操作系统会唤醒当前进程**，继续向下执行代码。

- `TCP`需要进行`3`次握手，所以`connect`至少需要`3`次网络传输过程
- 在发送少量数据时`$client->send`都是可以立即返回的。发送大量数据时，`socket`缓存区可能会塞满，`send`操作会阻塞。
- `recv`操作会阻塞等待服务器返回数据，`recv`耗时等于服务器处理时间+网络传输耗时之合。


# 创建异步`TCP`客户端

```
    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
    
    // 注册连接成功回调
    $client->on("connect", function ($cli) {
        $cli->send("hello world\n");
    });
    
    // 注册数据接收回调
    $client->on("receive", function ($cli, $data) {
        echo "Received: ".$data."\n";
    });
    
    // 注册连接失败回调
    $client->on("error", function ($cli) {
        echo "Connect failed\n";
    });
    
    // 注册连接关闭回调
    $client->on("close", function ($cli) {
        echo "Connection close\n";
    });
    
    // 发起连接
    $client->connect('127.0.0.1', 9501, 0.5);
```

> 异步客户端与上一个同步TCP客户端不同，**异步客户端是非阻塞的**。可以用于编写高并发的程序。
`swoole`官方提供的`redis-async`、`mysql-async`都是基于异步`swoole_client`实现的。

异步客户端需要设置回调函数，
有4个事件回调必须设置`onConnect`、`onError`、`onReceive`、`onClose`。
分别在客户端`连接成功`、`连接失败`、`收到数据`、`连接关闭`时触发。

`$client->connect()` 发起连接的操作会立即返回，不存在任何等待。
当对应的IO事件完成后，`swoole`底层会自动调用设置好的回调函数。

> 异步客户端只能用于`cli`环境

# 网络通信协议设计

## 为什么需要通信协议?

> TCP协议在底层机制上解决了UDP协议的顺序和丢包重传问题。
但相比UDP又带来了新的问题，**TCP协议是流式的**，数据包没有边界。应用程序使用TCP通信就会面临这些难题。

因为TCP通信是流式的，在接收1个大数据包时，可能会被拆分成多个数据包发送。
多次Send底层也可能会合并成一次进行发送。这里就需要2个操作来解决：

- `分包`：`Server`收到了多个数据包，需要拆分数据包
- `合包`：`Server`收到的数据只是包的一部分，需要缓存数据，合并成完整的包

所以TCP网络通信时需要设定通信协议。
常见的**TCP网络通信协议**有
- `HTTP`、`HTTPS`、`FTP`、`SMTP`、`POP3`、`IMAP`、`SSH`、`Redis`、`Memcache`、`MySQL` 。

如果要设计一个通用协议的Server，那么就要按照通用协议的标准去处理网络数据。
除了通用协议外还可以自定义协议。**Swoole支持了2种类型的自定义网络通信协议。**

### EOF结束符协议

> EOF协议处理的原理是每个数据包结尾加一串特殊字符表示包已结束。
如`memcache`、`ftp`、`stmp`都使用`\r\n`作为结束符。发送数据时只需要在包末尾增加`\r\n`即可。
使用EOF协议处理，一定要确保数据包中间不会出现EOF，否则会造成分包错误。

在`swoole_server`和`swoole_client`的代码中只需要设置2个参数就可以使用EOF协议处理。

```
    $server->set(array(
        'open_eof_split' => true,
        'package_eof' => "\r\n",
    ));
    
    $client->set(array(
        'open_eof_split' => true,
        'package_eof' => "\r\n",
    ));
```

### 固定包头+包体协议

固定包头的协议非常通用，在BAT的服务器程序中经常能看到。这种协议的特点是**一个数据包总是由包头+包体2部分组成**

> 包头由一个字段指定了包体或整个包的长度，长度一般是使用2字节/4字节整数来表示。
服务器收到包头后，可以根据长度值来精确控制需要再接收多少数据就是完整的数据包。

`Swoole`的配置可以很好的支持这种协议，可以灵活地设置4项参数应对所有情况。

Swoole的`Server`和`异步Client`都是在`onReceive`回调函数中处理数据包，
当设置了协议处理后，**只有收到一个完整数据包时才会触发`onReceive`事件**

同步客户端在设置了协议处理后，调用 `$client->recv()` 不再需要传入长度，`recv`函数在收到完整数据包或发生错误后返回。

```
    $server->set(array(
        'open_length_check' => true,
        'package_max_length' => 81920,
        'package_length_type' => 'n', //see php pack()
        'package_length_offset' => 0,
        'package_body_offset' => 2,
    ));
```

# 使用异步客户端

**PHP提供的MySQL、CURL、Redis 等客户端是同步的，会导致服务器程序发生阻塞。**
Swoole提供了常用的异步客户端组件，来解决此问题。编写纯异步服务器程序时，可以使用这些异步客户端。

> 异步客户端可以配合使用`SplQueue`实现连接池，以达到长连接复用的目的。
在实际项目中可以使用PHP提供的`Yield/Generator`语法实现半协程的异步框架。也可以基于`Promises`简化异步程序的编写。

## MySQL

```
    $db = new Swoole\MySQL;
    
    $server = array(
        'host' => '127.0.0.1',
        'user' => 'test',
        'password' => 'test',
        'database' => 'test',
    );
    
    $db->connect($server, function ($db, $result) {
        $db->query("show tables", function (Swoole\MySQL $db, $result) {
            var_dump($result);
            $db->close();
        });
    });
```

与`mysqli`和`PDO`等客户端不同，`Swoole\MySQL`是**异步非阻塞**的，连接服务器、执行SQL时，需要传入一个回调函数。
`connect`的结果不在返回值中，而是在回调函数中。`query`的结果也需要在回调函数中进行处理。


## Redis

```
    $redis = new Swoole\Redis;
    
    $redis->connect('127.0.0.1', 6379, function ($redis, $result) {
        $redis->set('test_key', 'value', function ($redis, $result) {
            $redis->get('test_key', function ($redis, $result) {
               var_dump($result);
            });
        });
    });
```

## Http

```
    $cli = new Swoole\Http\Client('127.0.0.1', 80);
    $cli->setHeaders(array('User-Agent' => 'swoole-http-client'));
    $cli->setCookies(array('test' => 'value'));
    
    $cli->post('/dump.php', array("test" =>' abc'), function ($cli) {
        var_dump($cli->body);
        $cli->get('/index.php', function ($cli) {
            var_dump($cli->cookies);
            var_dump($cli->headers);
        });
    });
```

`Swoole\Http\Client`的作用与`CURL`完全一致，它完整实现了`Http客户端`的相关功能。具体请参考 [HttpClient文档](https://wiki.swoole.com/wiki/page/p-http_client.html)

## 其他客户端

`Swoole`底层目前只提供了最常用的`MySQL`、`Redis`、`Http异步客户端`，
如果你的应用程序中需要实现其他协议客户端，如`Kafka`、`AMQP`等协议，
可以基于`Swoole\Client异步TCP客户端`，开发相关协议解析代码，来自行实现


# 多进程共享数据

由于PHP语言不支持多线程，因此`Swoole`使用多进程模式。在多进程模式下存在进程内存隔离，
在工作进程内修改`global全局变量`和`超全局变量`时，在其他进程是无效的。

## 进程隔离

```
    $fds = array();
    $server->on('connect', function ($server, $fd){
        echo "connection open: {$fd}\n";
        global $fds;
        $fds[] = $fd;
        var_dump($fds);
    });
```

`$fds` 虽然是全局变量，但**只在当前的进程内有效**。
`Swoole`服务器底层会创建多个`Worker`进程，在`var_dump($fds)`打印出来的值，只有部分连接的`fd`。

对应的解决方案就是使用外部存储服务：

- 数据库，如：`MySQL`、`MongoDB`
- 缓存服务器，如：`Redis`、`Memcache`
- 磁盘文件，多进程并发读写时需要加锁

普通的数据库和磁盘文件操作，存在较多IO等待时间。因此推荐使用：

- `Redis` 内存数据库，读写速度非常快
- `/dev/shm` 内存文件系统，读写操作全部在内存中完成，**无IO消耗，性能极高**

除了使用存储之外，还可以使用共享内存来保存数据

# 参考

- [快速起步](https://wiki.swoole.com/wiki/page/p-quickstart.html)