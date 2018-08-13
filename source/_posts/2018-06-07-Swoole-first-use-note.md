---
title: Swoole 初步了解 
date: 2018-06-07 14:22:10
categories: Swoole
tags: [Swoole]
---

# `Swoole` 初步了解

> `Swoole`这个名字不是一个英文单词，是由我创造的一个音近字。
> 我最早想到的名字是叫做`sword-server`，寓意是为广大PHPer创造一把锋利的剑，
> 后来联想到`google`也是凭空创造出来的，所以我就给它命名为`swoole`。 <韩天峰>

<!-- more -->

# 安装

- `PECL` 一键安装

```
    pecl install swoole
```

- 源码安装

```
    // 下载源码并解压
    cd swoole-src-swoole-1.8.0-table/
    phpize
    ./configure --enable-async-mysql
    sudo make
    sudo make install
```

## 配置 `php.ini`

```
    extension=swoole.so
```

# 基础实例

## 服务端

```
    class Server
    {
        private $serv;
    
        public function __construct() {
            $this->serv = new swoole_server("0.0.0.0", 9501);
            $this->serv->set(array(
                'worker_num'    => 8,
                'daemonize'     => false,
                'max_request'   => 10000,
                'dispatch_mode' => 2,
                'debug_mode'    => 1
            ));
    
            $this->serv->on('Start', array($this, 'onStart'));
            $this->serv->on('Connect', array($this, 'onConnect'));
            $this->serv->on('Receive', array($this, 'onReceive'));
            $this->serv->on('Close', array($this, 'onClose'));
    
            $this->serv->start();
        }
        
        public function onStart( $serv ) {
            echo "Start\n";
        }
    
        public function onConnect( $serv, $fd, $from_id ) {
            $serv->send( $fd, "Hello {$fd}!" );
        }
    
        public function onReceive( swoole_server $serv, $fd, $from_id, $data ) {
            echo "Get Message From Client {$fd}:{$data}\n";
        }
    
        public function onClose( $serv, $fd, $from_id ) {
            echo "Client {$fd} close connection\n";
        }
    }
    
    // 启动服务器
    $server = new Server();
```

## 解析

从代码中可以看出，创建一个`swoole_server`基本分三步：

1. 通过构造函数创建`swoole_server`对象
2. 调用**set**函数设置`swoole_server`的相关配置选项
3. 调用**on**函数设置相关回调函数

> - `onStart`回调在`server`运行前被调用，
> - `onConnect`在有新客户端连接过来时被调用，
> - `onReceive`函数在有数据发送到`server`时被调用
> - `onClose`在有客户端断开连接时被调用。

### 配置选项说明

- `worker_num`:     指定启动的`worker`进程数。

- `max_request`:    每个`worker`进程允许处理的最大任务数。

- `max_conn `:      服务器允许维持的最大`TCP`连接数

- `ipc_mode `:      设置进程间的通信方式。
  - `1` => 使用unix socket通信
  - `2` => 使用消息队列通信
  - `3` => 使用消息队列通信，并设置为争抢模式
  
- `dispatch_mode `: 指定数据包分发策略。
  - `1` => 轮循模式，收到会轮循分配给每一个`worker`进程
  - `2` => 固定模式，根据连接的文件描述符分配`worker`。这样可以保证同一个连接发来的数据只会被同一个`worker`处理
  - `3` => 抢占模式，主进程会根据`Worker`的忙闲状态选择投递，只会投递给处于闲置状态的`Worker`
  
- `task_worker_num `:   服务器开启的`task`进程数。
> 设置此参数后，必须要给`swoole_server`设置`onTask/onFinish`两个回调函数，否则启动服务器会报错。

- `task_max_request `:  每个`task`进程允许处理的最大任务数。

- `task_ipc_mode `:  设置`task`进程与`worker`进程之间通信的方式。

- `daemonize `:  设置程序进入后台作为守护进程运行。
> 说明：长时间运行的服务器端程序必须启用此项。如果不启用守护进程，当ssh终端退出后，程序将被终止运行。
启用守护进程后，标准输入和输出会被重定向到 `log_file`，如果 `log_file`未设置，则所有输出会被丢弃。

- `log_file`:   指定日志文件路径
> 说明：在`swoole`运行期发生的异常信息会记录到这个文件中。默认会打印到屏幕。
注意 `log_file` 不会自动切分文件，所以需要定期清理此文件。

- `heartbeat_check_interval `:  设置心跳检测间隔
> 说明：此选项表示每隔多久轮循一次，单位为秒。每次检测时遍历所有连接，
如果某个连接在间隔时间内没有数据发送，则强制关闭连接（会有`onClose`回调）。

- `heartbeat_idle_time`:  设置某个连接允许的最大闲置时间。
>说明：该参数配合`heartbeat_check_interval`使用。每次遍历所有连接时，如果某个连接在`heartbeat_idle_time`时间内没有数据发送，
则强制关闭连接。默认设置为`heartbeat_check_interval * 2`。

- `open_eof_check`: 打开eof检测功能
> 说明：与 `package_eof` 配合使用。此选项将检测客户端连接发来的数据，当数据包结尾是指定的`package_eof` 字符串时
才会将数据包投递至`Worker`进程，否则会一直拼接数据包直到缓存溢出或超时才会终止。
一旦出错，该连接会被判定为恶意连接，数据包会被丢弃并强制关闭连接。

- `package_eof`:  设置EOF字符串
> 说明：`package_eof`最大只允许传入`8`个字节的字符串

- `open_length_check `:  打开包长检测
> 说明：包长检测提供了`固定包头+包体`这种格式协议的解析，。
启用后，可以保证`Worker`进程`onReceive`每次都会收到一个完整的数据包。

- `package_length_offset `:  包头中第几个字节开始存放了长度字段
> 说明：配合`open_length_check`使用，用于指明长度字段的位置。

- `package_body_offset`:  从第几个字节开始计算长度
> 说明：配合`open_length_check`使用，用于指明包头的长度。

- `package_length_type `:  指定包长字段的类型
  - `s` => int16_t 机器字节序
  - `S` => uint16_t 机器字节序
  - `n` => uint16_t 大端字节序
  - `N` => uint32_t 大端字节序
  - `L` => uint32_t 机器字节序
  - `l` => int 机器字节序
> 说明：配合open_length_check使用，指定长度字段的类型，参数如下：

- `package_max_length `:  设置最大数据包尺寸
> 说明：该值决定了数据包缓存区的大小。如果缓存的数据超过了该值，
则会引发错误。具体错误处理由开启的协议解析的类型决定。

- `open_cpu_affinity `:  启用`CPU`亲和性设置
> 说明：在多核的硬件平台中，启用此特性会将`swoole`的`reactor`线程/`worker`进程绑定到固定的一个核上。
可以避免进程/线程的运行时在多个核之间互相切换，提高`CPU Cache`的命中率。

- `open_tcp_nodelay `:  启用`open_tcp_nodelay`
> 说明：开启后`TCP`连接发送数据时会无关闭`Nagle`合并算法，
立即发往客户端连接。在某些场景下，如`http`服务器，可以提升响应速度。

- `tcp_defer_accept`:  启用`tcp_defer_accept`特性
> 说明：启动后，只有一个`TCP`连接有数据发送时才会触发`accept`。

- `ssl_cert_file`/`ssl_key_file `:  设置SSL隧道加密
> 说明：设置值为一个文件名字符串，指定`cert`证书和`key`的路径。

- `open_tcp_keepalive `:  打开`TCP`的`KEEP_ALIVE`选项
> 说明：使用TCP内置的`keep_alive`属性，用于保证连接不会因为长时闲置而被关闭。

- `tcp_keepidle `:  指定探测间隔。
> 说明：配合`open_tcp_keepalive`使用，如果某个连接在`tcp_keepidle`内没有任何数据来往，则进行探测。

- `tcp_keepinterval `:  指定探测时的发包间隔
> 说明：配合`open_tcp_keepalive`使用

- `tcp_keepcount `:  指定探测的尝试次数
> 说明：配合`open_tcp_keepalive`使用，若`tcp_keepcount`次尝试后仍无响应，则判定连接已关闭

- `backlog `:  指定`Listen`队列长度
> 说明：此参数将决定最多同时有多少个等待`accept`的连接。

- `reactor_num`:  指定`Reactor`线程数
> 说明：设置主进程内事件处理线程的数量，默认会启用CPU核数相同的数量， 
一般设置为CPU核数的1-4倍，最大不得超过CPU核数*4。

- `task_tmpdir`:  设置task的数据临时目录
> 说明：在`swoole_server`中，如果投递的数据超过`8192`字节，
将启用临时文件来保存数据。这里的`task_tmpdir`就是用来设置临时文件保存的位置。

## 客户端

```
    <?php
    class Client
    {
    	private $client;
    
    	public function __construct() {
    		$this->client = new swoole_client(SWOOLE_SOCK_TCP);
    	}
    	
        public function connect() {
            if( !$this->client->connect("127.0.0.1", 9501 , 1) ) {
                echo "Error: {$fp->errMsg}[{$fp->errCode}]\n";
            }
            $message = $this->client->recv();
            echo "Get Message From Server:{$message}\n";
    
            fwrite(STDOUT, "请输入消息：");  
            $msg = trim(fgets(STDIN));
            $this->client->send( $msg );
        }
    }
    
    $client = new Client();
    $client->connect();
```

> 这里，通过`swoole_client`创建一个基于`TCP`的客户端实例，
并调用`connect`函数向指定的`IP`及端口发起连接请求。随后即可通过`recv()`和`send()`两个函数来接收和发送请求。
需要注意的是，这里我使用了默认的**同步阻塞客户端**，因此`recv`和`send`操作都会产生网络阻塞。


# `Swoole` 异步任务`Task`

## `Task`简介

`Swoole`的业务逻辑部分是同步阻塞运行的，如果遇到一些耗时较大的操作，例如访问数据库、广播消息等，就会影响服务器的响应速度。
因此`Swoole`提供了`Task`功能，将这些耗时操作放到另外的进程去处理，当前进程继续执行后面的逻辑。

## 开启`Task`功能

开启`Task`功能只需要在`swoole_server`的配置项中添加`task_worker_num`一项即可，如下：

```
    $serv->set(array(
        'task_worker_num' => 8
    ));
``` 

即可开启`task`功能。此外，必须给`swoole_server`绑定两个回调函数：`onTask`和`onFinish`。
这两个回调函数分别用于**执行`Task`任务**和**处理`Task`任务**的返回结果。

## 使用`Task`首先是发起一个`Task`，代码如下：

```
    public function onReceive (swoole_server $serv, $fd, $from_id, $data ) {
        echo "Get Message From Client {$fd}:{$data}\n";
        // send a task to task worker
        $parm = array(
            'fd' => $fd
        );
        
        // start a task
        $serv->task(json_encode($param));
        
        echo "Continue Handle Worker\n";
    }    
```

可以看到，发起一个任务时，只需通过`swoole_server`对象调用`task`函数即可发起一个任务。
`swoole`内部会将这个请求投递给`task_worker`，而当前`Worker`进程会继续执行。

当一个任务发起后，`task_worker`进程会响应`onTask`回调函数，如下：

```
    public function onTask($serv, $task_id, $from_id, $data)
    {
        echo "This Task {$task_id} from Worker {$from_id}\n";
        echo "Data: {$data}\n";
        
        for($i = 0; $i < 10; $i ++) {
            sleep(1);
            echo "Task {$task_id} Handle {$i} times...\n";
        }
        $fd = json_decode($data, true)['fd'];
        $serv->send($fd, "Data in Task {$task_id}");        
        return "Task {$task_id}'s result";
    }
```

> 这里我用`sleep`函数和循环模拟了一个长耗时任务。在`onTask`回调中，我们通过`task_id`和`from_id`(也就是`worker_id`)来区分不同进程投递的不同`task`。
当一个`task`执行结束后，通过`return`一个字符串将执行结果返回给`Worker`进程。
`Worker`进程将通过`onFinish`回调函数接收这个处理结果。

```
    public function onFinish($serv, $task_id, $data)
    {
        echo "Task {$task_id} finish\n";
        echo "Result: {$data}\n";
    }
```

在`onFinish`回调中，会接收到`Task`任务的处理结果`$data`。在这里处理这个返回结果即可。


## `swoole_client`

> 在这里讲解如何使用`swoole_client`是因为，在写服务端代码的时候，不可避免的需要用到客户端来进行测试。
`swoole`提供了`swoole_client`用于编写测试客户端，下面我将讲解如何使用这个工具。

`swoole_client`有两种工作模式：**同步阻塞模式**和**异步回调模式**

其中，同步阻塞模式在上一章中已经给出示例，其使用和一般的`socket`基本无异。
因此，我将重点讲解`swoole_client`的**异步回调模式**。

创建一个异步client的代码如下：

```
    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
``` 

其中，`SWOOLE_SOCK_ASYNC`选项即表明创建一个异步`client`。

既然是异步，那当然需要回调函数。`swoole_client`一共有四个回调函数，如下：

```
    $client->on("connect", function ($cli) {
        $cli->send("hello world\n");
    });
    $client->on("receive", function ($cli, $data) {
        echo "Received: ".$data."\n";
    });
    $client->on("error", function ($cli) {
        echo "Connect failed\n";
    });
    $client->on("close", function ($cli) {
        echo "Connection close\n";
    });
```

这几个回调函数的作用基本和`swoole_server`类似，只有参数不同，因此不再赘述。

# `Timer`计时器、心跳检测、`Task`进阶


## `Timer定时器`

> 在实际应用中，往往会遇到需要每隔一段时间重复做一件事，比如心跳检测、订阅消息、数据库备份等工作。
通常，我们会借助`PHP的time()`以及相关函数自己实现一个定时器，或者使用`crontab`工具来实现。
但是，自定义的定时器容易出错，而使用`crontab`则需要编写额外的脚本文件，无论是迁移还是调试都比较麻烦。
>
> 因此，`Swoole`提供了一个内置的`Timer`定时器功能,通过函数`addtimer`即可在`Swoole`中添加一个定时器，
该定时器会在建立之后，按照预先设定好的时间间隔，每到对应的时间就会调用一次回调函数`onTimer`通知`Server`。

```
    <?php
    
    $this->serv->on('Timer', array($this, 'onTimer'));
    
    public function onWorkerStart($serv, $worker_id)
    {
        // 在Worker进程开启时绑定定时器
        // 只有当worker_id为0时才添加定时器,避免重复添加
        if ($worker_id == 0) {
            $serv->addTimer(500);
            $serv->addTimer(1000);
            $serv->addTimer(1500);
        }
    }
    
    public function onTimer($serv, $interval)
    {
        switch ($interval) {
            case 500: {
                echo "Do Thing A at interval 500\n";
                break;
            }
            case 1000: {
                echo "Do Thing B at interval 1000\n";
                break;
            }
            case 1500: {
                echo "Do Thing C at interval 1500\n";
                break;
            }
        }
    }
```

可以看到，在`onWorkerStart`回调函数中，通过`addtimer`添加了三个定时器，时间间隔分别为`500`、`1000`、`1500`。
而在`onTimer`回调中，正好通过间隔的不同来区分不同的定时器回调，从而执行不同的操作。

> 需要注意的是，在上述示例中，**当1000ms的定时器被触发时，500ms的定时器同样会被触发**，
但是不能保证会在1000ms定时器前触发还是后触发，因此需要注意，定时器中的操作不能依赖其他定时器的执行结果。

## 心跳检测

使用`Timer`定时器功能可以实现发送心跳包的功能。事实上，`Swoole`已经内置了心跳检测功能，能自动`close`掉长时间没有数据来往的连接。
而开启心跳检测功能，只需要设置`heartbeat_check_interval`和`heartbeat_idle_time`即可。如下：

```swoole
    $this->serv->set(
        array(
            'heartbeat_check_interval' => 60,
            'heartbeat_idle_time' => 600,
        )
    );    
```

## `Task`进阶：`MySQL`连接池

> 在`PHP`中，访问`MySQL`数据库往往是性能提升的瓶颈。而`MySQL`连接池我想大家都不陌生，这是一个很好的提升数据库访问性能的方式。
传统的`MySQL`连接池，是预先申请一定数量的连接，每一个新的请求都会占用其中一个连接，请求结束后再将连接放回池中，
如果所有连接都被占用，新来的连接则会进入等待状态。
>
> 知道了`MySQL`连接池的实现原理，那我们来看如何使用`Swoole`实现一个连接池。
首先，`Swoole`允许开启一定量的`Task Worker`进程，我们可以让每个进程都拥有一个`MySQL`连接，并保持这个连接，这样，我们就创建了一个连接池。
> 
> 其次，设置`swoole`的`dispatch_mode`为**抢占模式**(主进程会根据`Worker`的忙闲状态选择投递，
只会投递给处于闲置状态的`Worker`)。这样，每个`task`都会被投递给闲置的`Task Worker`。
这样，我们保证了每个新的`task`都会被闲置的`Task Worker`处理，如果全部`Task Worker`都被占用，则会进入等待队列。

下面直接上关键代码：

```php
    public function onWorkerStart($serv, $worker_id)
    {
        echo "onWorkerStart\n";
    
        // 判定是否为Task Worker进程
        if ($worker_id >= $serv->setting['worker_num']) {
            $this->pdo = new PDO(
                "mysql:host=localhost;port=3306;dbname=Test",
                'root',
                '123456',
                array(
                    PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES 'UTF8';",
                    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                    PDO::ATTR_PERSISTENT => true
                )
            );
        }
    }
```

首先，在每个`Task Worker`进程中，创建一个`MySQL`连接。这里我选用了`PDO`扩展。

```mysql
    public function onReceive(swoole_server $serv, $fd, $from_id, $data) {
        $sql = array(
            'sql'  => 'select * from Test where pid > ?',
            'parm' => array(0),
            'fd'   => $fd
        );
    
        $serv->task(json_encode($sql));
    }
```

其次，在需要的时候，通过task函数投递一个任务（也就是发起一次`SQL`请求）

```
   public function onTask($serv,$task_id,$from_id, $data)
   {
      	$sql = json_decode( $data , true );
   	
        $statement = $this->pdo->prepare($sql['sql']);
        $statement->execute($sql['param']);    	
   
        $result = $statement->fetchAll(PDO::FETCH_ASSOC);
        $serv->send( $sql['fd'],json_encode($result));
   	
   	    return true;
   } 
```

> 最后，在`onTask`回调中，根据请求过来的SQL语句以及相应的参数，发起一次`MySQL`请求，
并将获取到的结果通过`send`发送给客户端（或者通过`return`返回给`Worker`进程）。
而且，这样的一次`MySQL`请求还不会阻塞`Worker`进程，`Worker`进程可以继续处理其他的逻辑。
 

# `Swoole`多端口监听、热重启以及`Timer`进阶：简单`crontab`

## 多端口监听

```
    $serv = new swoole_server("192.168.1.1", 9501);     // 监听外网的9501端口
    $serv->addlistener("127.0.0.1", 9502 , SWOOLE_TCP); // 监听本地的9502端口
    
    $serv->start(); // addlistener必须在start前调用
```

> 此时，`swoole_server`就会同时监听两个`host`下的两个端口。这里要注意的是，
来自两个端口的数据会在同一个`onReceive`中获取到，这时就要用到`swoole`的另一个成员函数`connection_info`，
通过这个函数获取到`fd`的`from_port`，就可以判定消息的类型

```
    $info = $serv->connection_info($fd, $from_id);
    //来自9502的内网管理端口
    if($info['from_port'] == 9502) {
        $serv->send($fd, "welcome admin\n");
    }
    //来自外网
    else {
        $serv->send($fd, 'Swoole: '.$data);
    }
```

## 服务器热重启

> 所谓热重启，就是当服务器相关代码有所变动之后，无需停止服务，而是在服务器仍然运行的状态下更新文件。

`Swoole`通过内置的`reload`函数以及两个自定义信号量实现了这一功能

`Swoole`可用的三个信号：`SIGTERM`，`SIGUSR1`，`SIGUSR2`

- `SIGTERM`用于停止服务器
- `SIGUSR1`用于重启全部的`Worker`进程
- `SIGUSR2`用于重启全部的`Task Worker`进程

### 那要如何实现热更新代码文件呢？

Swoole的回调函数中有这个一个回调`onWorkerStart`;该回调会在`Worker`进程启动时被调用。
因此，当`swoole_server`收到`SIGUSR1`信号并重启全部`Worker`进程后，
`onWorkerStart`就会被调用。如果在`onWorkerStart`中`require`全部的代码文件，
每次`onWorkerStart`后都会重新`require`一次php文件，这样就能实现代码文件的热更新。

```
    public function onStart( $serv )
    {
            cli_set_process_title("reload_master");
        }
    public function onWorkerStart( $serv , $worker_id)
    {
        require_once "reload_page.php";
        Test(); // reload_page.php中定义的一个函数
    }
```

首先，在`onStart`回调函数中通过php的`cli_set_process_title`函数设置进程名。 
在`onWorkerStart中`，`require`相关的php文件。 然后，新建一个`reload.sh`文件，输入如下内容：

```
    echo "Reloading..."
    cmd=$(pidof reload_master)
    
    kill -USR1 "$cmd"
    echo "Reloaded"
```

这样，就可以通过执行这个脚本重启服务器了 

## `Timer`补充：`after`函数

在`swoole-1.7.7stable`版本中，`Timer`新增了一个函数`after`。
该函数的作用是在指定的时间间隔后执行回调函数，并且只执行一次。

```
    $serv->after( 1000 , array($this, 'onAfter') , $str );
```

这里指定在`1000ms`后，执行`onAfter`回调函数，函数参数为`$str`

# `Swoole`的自定义协议功能的使用

## 为什么要提供自定义协议 ?

熟悉TCP通信的朋友都会知道，**TCP是一个流式协议**。客户端向服务器发送的一段数据，可能并不会被服务器一次就完整的收到;
客户端向服务器发送的多段数据，可能服务器一次就收到了全部的数据。

而实际应用中，我们希望在服务器端能一次接收一段完整的数据，不多也不少

传统的TCP服务器中，往往需要由程序员维护一个缓存区，先将读到的数据放进缓存区中，然后再通过预先设定好的协议内容，来区分一段完整数据的开头、长度和结尾，
并将一段完整的数据交给逻辑部分处理。这就是自定义协议的功能。

而在`Swoole`中，已经在底层实现了一个数据缓存区，并内置好了几个常用的协议类型，
直接在底层做好了数据的拆分，保证了在`onReceive`回调函数中，一定能收到一个（或数个）完整的数据段。
数据缓存区的大小可以通过配置选项`package_max_length`来控制。

下面我就将讲解如何使用这些内置协议。

## `EOF`标记型协议

> 第一个比较常用的协议就是EOF标记协议。协议的内容是通过规定一个一定不会出现在正常数据中的字符或者字符串，
用这个来标记一段完整数据的结尾
> 这样，只要发现这个结尾，就可以认定之前的数据已经结束，可以开始接收一个新的数据段了。

在`Swoole`中，可以通过`open_eof_check`和`package_eof`两个配置项来开启。
其中，`open_eof_check`指定开启了`EOF`检测，`package_eof`指定了具体的`EOF`标记

通过这两个选项，Swoole底层就会自动根据EOF标记来缓存和拆分收到的数据包

```
    $this->serv->set(array(
        'package_max_length => 8192,
        'open_eof_check'    => true,
        'package_eof'       => "\r\n"
    ));
```

就这样，`swoole`就已经开启了**EOF标记协议**的解析。那么让我们来测试一下效果：

- 服务器

```
    // server
    public function onReceive(swoole_serve $serv, $fd, $from_id, $data)
    {
        echo "Get Message From Client {$fd}:{$data}\n";
    }
```

- 客户端

```
    $msg_eof = "This is a Msg\r\n";
    
    $i = 0;
    while($i < 100) {
        $this->client->send($msg_eof);
        $i ++;
    }
```

> 然后运行一下，你会发现：哎不对啊，为什么还是一次收到了好多数据啊！
  这是因为，在Swoole中，采用的不是遍历识别的方法，而只是简单的检查每一次接收到的数据的末尾是不是定义好的EOF标记。因此，在开启EOF检测后，onReceive回调中还是可能会一次收到多个数据包。
  这要怎么办？你会发现，虽然是多个数据包，但是实际上收到的是N个完整的数据片段，那就只需要根据EOF把每个包再拆出来，一个个处理就好啦。
  
- 修改后的服务器端代码如下：

```
    // server
    public function onReceive(swoole_serve $serv, $fd, $from_id, $data)
    {
        $data_list = explode("\r\n", $data);
        foreach($data_list as $msg) {
            if (!empty($msg) {
        echo "Get Message From Client {$fd}:{$data}\n";
            }
        }
    }
```

## 固定包头类型协议

固定包头协议是在实际应用中最常用的协议。
该协议的内容是**规定一个固定长度的包头**，在包头的固定位置有一个指定好的字段存放了后续数据的实际长度
这样，服务器端可以先读取固定长度的数据，从中提取出长度，然后再读取指定长度的数据，即可获取一段完整的数据。

在Swoole中，同样提供了固定包头的协议格式。
需要注意的是，`Swoole`只允许二进制形式的包头，因此，需要使用`pack`、`unpack`来打包、解包。

> 通过设置`open_length_check`选项，即可打开固定包头协议解析功能
。此外还有`package_length_offset`，`package_body_offset`和`package_length_type`三个配置项用于控制解析功能。
> - `package_length_offset`规定了包头中第几个字节开始是长度字段，
> - `package_body_offset`规定了包头的长度，
> - `package_length_type`规定了长度字段的类型。

```
    $this->serv->set(array(
        'package_max_length'    => 8192,
        'open_length_check'     => true,
        'package_length_offset' => 0,
        'package_body_offset'   => 4,
        'package_length_type'   => 'N'
    ));
```

### 实例

- 服务器端

```
    public function onReceive(swoole_server $serv, $fd, $from_id, $data)
    {
        $length = unpack("N", $data)[1];
        echo "Length = {$length}\n";
        $msg = substr($data, -$length);
        
        echo "Get Message From Client {$fd}:{$msg}\n";
    }
```

- 客户端

```
    $msg_lenght = pack("N", strlen($msg_normal)).$msg_normal;
    
    $i = 0;
    while($i < 100) {
        $this->client->send($msg_lengt);
        $i++;
    }
```

## 特别篇：`Http`协议-`Swoole`内置的`http_server`

 从`Swoole-1.7.7-stable`开始，`Swoole`在内部封装并实现了一个`Http`服务器。
 是的，没错，再也不用在`PHP`层缓存和解析`http`协议了，`Swoole`直接内置`Http`服务器了。
 
 - `swoole_http_server`
 
 ```
    $http = new swoole_http_server("127.0.0.1", 9501);
    $http->on('request', function (swoole_http_request $request, swoole_http_response $response) {
        $response->end("<h1>Hello Swoole.</h1>");
    });
    $http->start();
 ```

> 只需创建一个`swoole_http_server`对象并设置`onRequest`回调函数，即可实现一个`http`服务器。

在onRequest回调中，共有两个参数。

- 参数`$request`存放了来自客户端的请求，包括`Http`请求的头部信息、`Http`请求相关的服务器信息、`Http`请求的`GET`和`POST`参数以及`HTTP`请求携带的`COOKIE`信息。
- 参数`$response`用于发送数据给客户端，可以通过该参数设置HTTP响应的`Header`信息、`cookie`信息和状态码。

此外，`swoole_http_server`还提供`WebSocket`功能，使用此功能需要设置`onMessage`回调函数，如下：

```
    $http_server->on('message', function(swoole_http_request $request, swoole_http_response $response) {
        echo $request->message;
        $response->message(json_encode(array("data1", "data2")));
    })
```

通过`$request->message`获取`WebSocket`发送来的消息，再通过`$response->message()`回复消息即可。

# 参考

- [Swoole文档](https://wiki.swoole.com/wiki/page/p-originate.html)
- [Swoole入门教程及文档](https://github.com/LinkedDestiny/swoole-doc)
- [swoole_server配置选项](https://github.com/LinkedDestiny/swoole-doc/blob/master/doc/01.swoole_server%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9.md)
