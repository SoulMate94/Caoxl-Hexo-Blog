---
title: PHP HTTP客户端-Guzzle原理解析
date: 2018-01-31 17:33:51
categories: PHP
tags: [PHP, HTTP, Client, Guzzle]
---

本文适合寻找PHP HTTP客户端库，或者对于Guzzle的使用和实现原理比较感兴趣的同学阅读，需要具备一定的PHP基础知识。

> Guzzle是一个PHP的HTTP客户端，用来轻而易举地发送请求，并集成到我们的WEB服务上。

<!--more-->

# 背景

在PHP后台开发过程中，经常会遇到模块间需要通过HTTP通信的情形。PHP语言本身只提供了`socket`操作的接口，并未提供HTTP相关操作的接口。许多现有的实现采用curl扩展充当 `HTTP Client` 与 `HTTP Server` 通信，但仍需自己封装curl的接口。有鉴于此，本文介绍一款流行的 PHP HTTP Client客户端---<u>[Guzzle](https://github.com/guzzle/guzzle/)</u>的用法，深入分析其底层实现原理。

# `Guzzle`基础

## 需求

- PHP 5.5.0+
- 使用PHP的流， `allow_url_fopen` 必须在**php.ini**中启用。
- 要使用cURL，你必须已经有版本 `cURL >= 7.19.4`，并且编译了 `OpenSSL` 与 `zlib`

## 安装

你可以使用composer.phar客户端将Guzzle作为依赖添加到项目：

```
    php composer.phar require guzzlehttp/guzzle:~6.0
```

或者，你可以编辑项目中已存在的composer.json文件，添加Guzzle作为依赖：

```
    {
       "require": {
          "guzzlehttp/guzzle": "~6.0"
       }
    }
```

安装完毕后，你需要引入Composer的自动加载文件：

```
    require 'vendor/autoload.php';
```

## 测试

```
    git clone https://github.com/guzzle/guzzle.git
    cd guzzle && curl -s http://getcomposer.org/installer | php && ./composer.phar install --dev
```

Guzzle is unit tested with PHPUnit. Run the tests using the Makefile:

```
    make test
```

# `Guzzle`用法

例如使用Guzzle访问 `http://www.baidu.com` 的代码：

```
    <?php
    
    $client = new \GuzzleHttp\Client();
    
    $response = $client->request('GET', 'http://www.baidu.com', [
        "timeout" => 3000
    ]);
    
    echo $response->getStatusCode(), "\n";
    echo $response->getBody();
```

接口封装是不是十分简单？只需要关心请求方法，目标url和请求的选项即可快速上手。同时，`Guzzle`还支持异步请求方式：

```
    <?php
    
    use GuzzleHttp\Exception\RequestException;
    use Psr\Http\Message\ResponseInterface;
    
    $client = new \GuzzleHttp\Client();
    $promise = $client->requestAsync('GET', 'http://www.baidu.com');
    $promise->then(
        function (ResponseInterface $res) {
            echo $res->getStatusCode() . "\n";
            echo $res->getBody();
            return $res;
        },
        function (RequestException $e) {
            echo $e->getMessage() . "\n";
            echo $e->getRequest()->getMethod();
        }
    )->wait();
```

基于异步请求，Guzzle还实现了并发请求，关于Guzzle的具体使用方法可以参考其 <u>[Guzzle中文文档](http://guzzle-cn.readthedocs.io/zh_CN/latest/index.html)</u>


# `Guzzle`实现原理

## 1.client构造

`GuzzleHttp\Client`类构造函数声明为：

```
    public function __construct(array $config = [])
```

`$config`配置使得用户可以根据需要配置一切可以配置的选项，包括`allowredirects`、`auth`、`connecttimeout`、`proxy`等。除此之外，还可以自定义请求的处理函数`handler`，方便应用程序扩展，`handler`接口规范为：

```
    function handler($request, array $options);
```

处理成功时，接口返回 `Psr\Http\Message\ResponseInterface`；
失败时返回 `GuzzleHttp\Exception\RequestException`异常。

默认情形下，`GuzzleHttp\HandlerStack::create`会创建请求处理函数

```
    public static function create(callable $handler = null)
    {
        $stack = new self($handler ?: choose_handler());
        $stack->push(Middleware::httpErrors(), 'http_errors');
        $stack->push(Middleware::redirect(), 'allow_redirects');
        $stack->push(Middleware::cookies(), 'cookies');
        $stack->push(Middleware::prepareBody(), 'prepare_body');
    
        return $stack;
    }
```

`create`函数以堆栈的形式创建了一系列的处理函数，包括 **http异常**、**重定向**、**cookie**和**prepare_body**。处理函数返回的函数闭包为：

```
    return function (callable $handler) {
        return function ($request, array $options) use ($handler) {
            ...
        };
    };
```

函数入参为`handler`，返回一个新的`handler`，这样可以将所有的处理函数链接在一起，最终生成一个符合`handler`接口规范的函数.

`choose_handler`函数选择`stack`中的起始`handler`，选择策略为：

- 扩展自带`curl_multi_exec`和`curl_exec`函数则根据`$options`中的`synchronous`选项决定，`empty(synchronous)`为 `false`则使用`CurlHandler`，否则使用`CurlMultiHandler`

- 扩展只有`curl_exec`函数则使用`CurlHandler`

- 扩展只有`curl_multi_exec`函数则使用`CurlMultiHandler`

最后，如果 **php.ini** 中开启了`allow_url_fopen`，则根据`$options`中的`stream`选项决定，`empty(stream)`为`false`则使用`StreamHandler`

## 2.client调用request方法

request方法实现为：

```
    public function request($method, $uri = '', array $options = [])
    {
        $options[RequestOptions::SYNCHRONOUS] = true;
        return $this->requestAsync($method, $uri, $options)->wait();
    }
```

由此可见，**request事实上是采用了requestAsync异步方法+wait来完成的，也就是异步转同步**。

### requestAsync

`requestAsync`将请求信息包装成 `Psr7\Request` 对象，然后调用

```
    transfer(RequestInterface $request, array $options)
```

`transfer`函数最终返回`Promise\promise_for($handler($request, $options))`; 其中`$handler`即为构造函数中所设置的`stack`，`stack`中存放一系列的请求处理函数。 `HandlerStack`的处理函数为:

```
    public function __invoke(RequestInterface $request, array $options)
    {
        $handler = $this->resolve();
    
        return $handler($request, $options);
    }
```

`resolve`方法解析整个`stack`，返回一个包装后的`handler`，包装策略为按照出栈顺序包装，也就是

```
    foreach (array_reverse($this->stack) as $fn) {
        $prev = $fn[0]($prev);
    }
```

![中间件模型](http://mmbiz.qpic.cn/mmbiz_png/Izj5340ib4qMJZfUdFvgicNGAMYhiccsYPv85otsPMkB0Gic6chnv4MMyMrfFZ10pDv9gpRFK9IznmeScvbdJhueZA/0?wx_fmt=png)

典型的中间件模型，所有的处理函数串接在一起了。请求经由`http_errors`、`allow_redirects`等处理之后到达`Curl`，执行真正的网络交互。

- 对于**同步**的`handler`如`CurlHandler`，在此处会执行`curl_exec`发起请求，最终返回的是`FulfilledPromise对象`或`RejectedPromise对象`，代表请求已经处理完毕。

- 对于**异步**的`handler`比如`CurlMultiHandler`，在此处并不会执行`curl_multi_exec`，而是返回一个`promise`对象，里面注册了需要等待执行的`curl_multi_exec`。

### wait

请求发送完毕，进入`promise`的`wait`操作，最终会执行`promise`的`$waitFn`函数。

- 对于`CurlMultiHandler`，`$waitFn`即执行`curl_multi_exec`进行网络交互，然后调用`resolve`方法将`response对象`传递到`then`方法的`$onFulfilled`函数。

- 对于`CurlHandler`，直接利用`resolve`将`response`对象传递到`$onFulfilled`函数。

这样，异步的`then`方法设置的回调就可以接收到`response`了。 **then方法最终返回response，这个对象又可以作为返回值返回，这样同步的wait就可以通过返回值来获取response对象了**。


# 总结

本文重点介绍了 `Guzzle` **同步**和**异步**请求的实现原理，除此之外，Guzzle还提供了**并行请求**，**请求pool**等实现，读者可以在此基础上继续深入。


# 参考

- [Guzzle中文文档](http://guzzle-cn.readthedocs.io/zh_CN/latest/index.html)

- [Guzzle, PHP HTTP client](https://github.com/guzzle/guzzle/)

- [Guzzle 快速入门](http://guzzle-cn.readthedocs.io/zh_CN/latest/quickstart.html)