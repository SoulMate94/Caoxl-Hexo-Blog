---
title: PHP异步的几个玩法
date: 2018-08-17 17:16:30
categories: PHP
tags: [PHP, 异步]
---

> 没有异步的PHP是世界上最好的语言, 拥有异步的PHP是宇宙上最好的语言

<!-- more -->

# 一. 先说说鸟哥文章中的几种玩法

- 一是通过渲染前端页面，**使用js执行Ajax**，这种方式现在还适用。只是受限于业务场景，因为只能在浏览器中调用，遇到接口请求就不行了。

- 二是通过`popen()`方法打开一个指向进程的管道，每个请求会多起一个进程。忽略进程来看最主要的原因是数据的传输特别不方便，使用场景有限。

- 三是使用`CURL`扩展，通过设置`timeout`超时参数，能实现离弦之箭的效果。不过这种方法会主动断开连接。被调用的服务如果有做连接检测，也会中断服务端脚本的执行。比如我们请求 微信的某个费时接口（20s），我们调用1s就断开连接，微信端是否会维持请求执行20S是不可控的。**所以这种方法不推荐大家使用**。

- 四方法与`CURL`类似，通过`fsockopen`创建`socket`连接访问远程服务，不循环获取请求结果。一样会有三中连接被断开的问题。


# 二. PHP发展了这么多年对异步支持方面都有哪些改进？

- `CURL`扩展已支持毫秒配置，将 `CURLOPT_TIMEOUT` 改为 `CURLOPT_TIMEOUT_MS `即可生效（`cURL` 版本 >= `libcurl`/`7.21.0`，老服务器要检查版本），但还是我前面说的需要服务端配合，不然接口的调用成功失败不可控。

- `CURL`扩展已支持并发，我们能一次访问N个接口，执行时间取最长接口的时间。比如我们能一次访问 京东支付（1s），微信支付(1.2s)，支付宝(0.8s)不同服务的三个接口,总耗时才1.2s。详细用法 [curl_multi_init](https://link.jianshu.com/?t=http://php.net/manual/en/function.curl-multi-init.php)

- 类似`Node.js`的异步IO框架`Swoole`，能很好的实现异步调用；不过`Swoole`理论上不能算PHP框架，他算是PHP功能的扩展。所以除非项目都用`Swoole`写，不然也是享受不到异步IO的福利。

- 对`yield`的支持，能实现调度器的功能，写单进程的服务时能大展拳脚，特别是实现协程，异步更不在话下。不过在多进程的web服务上没有太大的使用场景，看未来会不会有新的玩法吧。

> 当然还有很多新的特性，这里不再细说，总之PHP越是被黑越是能快速发展。

# 三. 最好的异步实现方法

我们都知道PHP是支持多进程编程的，那完全可以新建一个进程去实现异步的调用。比如调用`popen()`方法，但是管道的方式传参异常麻烦，不过多进程这个方法是绝对可行的。如果要实现多进程的功能，毫无疑问我们会选择PHP官方提供的 `pcntl` 扩展,PHP默认会安装 `pcntl` 扩展，

```
    <?php
    
    // @caoxl
    
    class Async
    {
        static $instance;
    
        public static function getInstance()
        {
            if (null == Async::$instance)
                Async::$instance = new Arrow();
            return Async::$instance;
        }
    
        public function run($rb)
        {
            $pid = pcntl_fork();
    
            if ($pid > 0) {
                pcntl_wait($status);
            } elseif ($pid == 0) {
                $cid = pcntl_fork();
                if ($cid > 0) {
                    exit();
                } elseif ($cid == 0) {
                    $rb();
                } else {
                    exit();
                }
            } else {
                exit();
            }
        }
    }
    
    $timeout = 30;
    Async::getInstance()->run(function () use ($timeout) {
        echo 2;
        sleep($timeout);
    });
    echo 1;
```

**代码说明:**

首先`Async`类是个单例类，减少多次调用的开销。`run()`方法传递的是一个匿名函数，这样我们能非常方便的传递参数，并且保留上下文。

这个类最难的地方在于多进程的处理。因为我们要尽可能快的将数据返回给用户，所以主进程越快结束越好。但是我们又需要子进程来执行我们耗时的操作，执行完退出才行。如果不等子进程执行完就将父进程退出会出现什么结果呢？结果就是子进程会常驻内存变成僵死进程。那我们有什么办法让子进程执行完之后就自动结束呢？答案是很难……那么儿子进程这么不听话，孙子进程会不会听话一点呢？？答案是孙子进程执行结束后会被系统进程回收并销毁（还是孙子听话）。所以我在代码中使用了如下方法：当前请求进程fork出子进程，子进程fork出孙子进程，主进程和子进程都先行退出，最后由孙子进程来执行耗时操作，最后完美的解决了僵死进程问题。

当然这个方法的缺点就是调用的时候会多产生一个`php-fpm`的进程

**为避免僵尸进程，当子进程结束后，手动杀死进程**

```
    <?php
    
    // PHP Linux Cli 模式下利用 pcntl_fork 实现多进程处理
    // @caoxl
    
    // 进程数
    $processes = 5;
    
    // 所有任务就是是为了输出0到99中的所有数字
    // 这些数字将被分成5个块，代表5个进程，
    // 当输出的时候我们就可以看到所有进程的执行顺序
    
    $tasks = range(0, 99);
    
    $blocks = array();
    
    // 将任务按进程分块
    
    foreach ($tasks as $i) {
        $blocks[($i % $processes)][] = $i;
    }
    
    foreach ($blocks as $blockNum => $block) {
        // 通过pcntl得到一个子进程的PID
        $pid = pcntl_fork();
    
        if ($pid == -1) {
            // 错误处理: 创建子进程失败时返回-1
            die('cloud not fork');
        } elseif ($pid) {
            // 父进程逻辑
            // 等待子进程中断, 防止子进程成为僵尸进程
            // WNOHANG为非阻塞进程，具体请查阅pcntl_wait PHP官方文档
            pcntl_wait($status, WNOHANG);
        } else {
            // 子进程逻辑
            foreach ($block as $i) {
                echo "I'm block {$blockNum}, I'm printing:{$i}\n";
                sleep(1);
            }
    
            // 为避免僵尸进程，当子进程结束后，手动杀死进程
            if (function_exists('posix_kill')) {
                posix_kill(getmypid(), SIGTERM);
            } else {
                system('kill -9'.getmypid());
            }
            exit();
        }
    
    }
```

# 参考

- [PHP实现异步调用方法研究](http://www.laruence.com/2008/04/14/318.html)
