---
title: Laravel / Lumen 生命周期
date: 2018-07-06 11:36:13
categories: Laravel
tags: [PHP, Laravel, Lumen]
---

> 当我们在决定使用某项技术的时候，除了需要了解它能「做什么」，其实还应当研究它是「怎么做的」。

<!-- more -->

# `Laravel` 生命周期

> 以下所有是基于 `Laravel-5.6.*`

> `Laravel` 的生命周期从`public\index.php`开始，从`public\index.php`结束。

## 生命周期始末

```php
    <?php
    
    // 第一阶段
    require __DIR__.'/../vendor/autoload.php';
    
    // 第二阶段
    $app = reuqire_once __DIR__.'/../bootstrap/app.php';
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    
    // 第三阶段
    $response = $kernel->handle(
        $request - Illuminate\Http\Request::capture()
    );
    
    // 发送响应
    $response->send();
    
    // 其它
    $kernel->terminate($request, $response);
```

## 不妨再详细一点

### 第一步: 加载项目依赖

> 现代 `PHP` 依赖于 `Composer` 包管理器，入口文件通过引入由 `Composer` 包管理器自动生成的类加载程序，
可以轻松注册并加载项目所依赖的第三方组件库。

所有组件的加载工作，仅需一行代码即可完成:

```php
    require __DIR__.'/../vendor/autoload.php';
```

### 第二步: 创建`Laravel`应用实例

下面是 `bootstrap/app.php` 的代码，包含两个主要部分`「创建应用实例」`和`「绑定内核至 APP 服务容器」`:

```php
    <?php
    
    // 第一部分: 创建应用实例
    $app = new Illuminate\Foundation\Application(
        realpath(__DIR__.'/../')
    );
    
    // 第二部分: 完成内核绑定
    $app->singleton(
        Illuminate\Contracts\Http\Kernel::class,
        App\Http\Kernel::class
    );
    
    $app->singleton(
        Illuminate\Contracts\Console\Kernel::class,
        App\Console\Kernel::class
    );
    
    // 第三部分: 注册异常处理
    $app->singleton(
        Illuminate\Contracts\Debug\ExceptionHandler::class,
        App\Exceptions\Handler::class
    );
    
    return $app;
```

#### 创建应用实例

创建应用实例即实例化 `Illuminate\Foundation\Application` 这个服务容器，后续我们称其为 `APP 容器`

在创建 `APP` 容器主要会完成: 
- 注册应用的基础路径并将路径绑定到 `APP` 容器 
- 注册基础服务提供者至 `APP` 容器 
- 注册核心容器别名至 `APP` 容器 等基础服务的注册工作。

```php
    public function __construct($basePath = null)
    {
        if ($basePath) {
            $this->setBasePath($basePath);
        }

        $this->registerBaseBindings();

        $this->registerBaseServiceProviders();

        $this->registerCoreContainerAliases();
    }
```

#### 内核绑定

`Laravel` 会依据 `HTTP` 请求的运行环境的不同，将请求发送至相应的内核: **HTTP 内核** 或 **Console 内核。无论 HTTP 内核还是 Console 内核**,
它们的作用都是是接收一个 `HTTP` 请求，随后返回一个响应，就是这么简单。


```php
    // Http
    $app->singleton(
        Illuminate\Contracts\Http\Kernel::class,
        App\Http\Kernel::class
    );
    
    // Console
    $app->singleton(
        Illuminate\Contracts\Console\Kernel::class,
        App\Console\Kernel::class
    );
```

##### 认识一下两个内核

请求被送到`HTTP`内核或`Console`内核, 这取决于进入应用的请求类型

> 取决于是通过浏览器请求还是通过控制台请求, 这里我们主要通过浏览器请求.

> `HTTP`内核的标志性方法`handle`处理的逻辑相当简单: 获取一个 `Request`,返回一个`Response`, 
把该内核想象作一个代表整个应用的大黑盒子, 输入`HTTP`请求, 返回`HTTP`响应.

- `HTTP`

```php
    protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];
```

- `Console`

```php
    protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class,
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class,
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class,
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class,
        \Illuminate\Foundation\Bootstrap\SetRequestForConsole::class,
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class,
        \Illuminate\Foundation\Bootstrap\BootProviders::class,
    ];
```

> 注意顺序:
`Facades` 先于 `ServiceProviders`, `Facades`也是重点,后面说.这里简单提一下,
注册`Facades`就是注册`config\app.php`中的`aliases`数组, 你使用的很多,如:
`Auth`, `Cache`, `DB`等等都是`Facades`, 而`ServiceProviders`的`register`方法永远先于
`boot`方法执行, 以免产生`boot`方法依赖某个实例而该实例还未注册的现象

> `HTTP`内核还定义了一系列所有请求在处理前需要经过的`HTTP`中间件, 这些中间件处理`HTTP`会话的读写、
判断应是否处于维护模式、验证`CSRF`令牌等等.


### 第三步: 接收请求并响应

在完成创建 `APP` 容器后即进入了第三个阶段 「接收请求并响应」。

**「接收请求并响应」** 有关代码如下:

```php
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );
    
    $response->send();
```

#### 解析内核实例

在第二步我们已经将 **HTTP 内核** 和 **Console 内核** 绑定到了 `APP` 容器，
使用时通过 `APP` 容器 的 `make()` 方法将内核解析出来，解析的过程就是内核实例化的过程。

```
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
```

##### 内核实例化时它的内部究竟又做了哪些操作呢？

进一步挖掘 `Illuminate\Foundation\Http\Kernel` 内核的 `__construct(IlluminateContractsFoundationApplication $app, \Illuminate\Routing\Router $router)` 构造方法,
它接收 **`APP` 容器** 和 **路由器** 两个参数。

在实例化内核时，构造函数内**将在 `HTTP` 内核定义的「中间件组」注册到 路由器**，
注册完后就可以在实际处理 `HTTP` 请求前调用这些「中间件」实现 **过滤** 请求的目的。

```php
    public function __construct(Application $app, Router $router)
    {
        $this->app = $app;
        $this->router = $router;

        $router->middlewarePriority = $this->middlewarePriority;

        foreach ($this->middlewareGroups as $key => $middleware) {
            $router->middlewareGroup($key, $middleware);
        }

        foreach ($this->routeMiddleware as $key => $middleware) {
            $router->aliasMiddleware($key, $middleware);
        }
    }
```

- `middlewareGroup`

```
   /**
    * Register a group of middleware.
    * 注册中间件组
    *
    * @param  string  $name
    * @param  array  $middleware
    * @return $this
    */
   public function middlewareGroup($name, array $middleware)
   {
       $this->middlewareGroups[$name] = $middleware;

       return $this;
   } 
```

- `aliasMiddleware`

```
    /**
     * Register a short-hand name for a middleware.
     * 注册中间件别名
     *
     * @param  string  $name
     * @param  string  $class
     * @return $this
     */
    public function aliasMiddleware($name, $class)
    {
        $this->middleware[$name] = $class;

        return $this;
    }
```

#### 处理`HTTP`请求

> 之前的所有处理，基本都是围绕在配置变量、注册服务等运行环境的构建上，构建完成后才是真刀真枪的来处理一个`「HTTP 请求」`。

处理请求实际包含两个阶段:
- 创建请求实例
- 处理请求

```
    // 处理请求
    $response = $kernel->handle(
        // 创建请求实例
        $request = Illuminate\Http\Request::capture();
    );
```

##### 创建请求实例

请求实例 `Illuminate\Http\Request` 的 `capture()` 方法内部通过 `Symfony` 实例创建一个 `Laravel` 请求实例,
这样我们就可以获取到用户请求报文的相关信息了

```
    /**
     * Create a new Illuminate HTTP request from server variables.
     *
     * @return static
     */
    public static function capture()
    {
        static::enableHttpMethodParameterOverride();

        return static::createFromBase(SymfonyRequest::createFromGlobals());
    }
    
    /**
     * Create an Illuminate request from a Symfony instance.
     *
     * @param  \Symfony\Component\HttpFoundation\Request  $request
     * @return \Illuminate\Http\Request
     */
    public static function createFromBase(SymfonyRequest $request)
    {
        if ($request instanceof static) {
            return $request;
        }

        $content = $request->content;

        $request = (new static)->duplicate(
            $request->query->all(), $request->request->all(), $request->attributes->all(),
            $request->cookies->all(), $request->files->all(), $request->server->all()
        );

        $request->content = $content;

        $request->request = $request->getInputSource();

        return $request;
    }
```

##### 处理请求

请求处理发生在 [HTTP](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Foundation/Http/Kernel.php)内核的`handle()`方法中

```
    /**
     * Handle an incoming HTTP request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
     public function handle($request)
     {
     try {
         $request->enableHttpMethodParameterOverride();

         $response = $this->sendRequestThroughRouter($request);
     } catch (Exception $e) {
         $this->reportException($e);

         $response = $this->renderException($request, $e);
     } catch (Throwable $e) {
         $this->reportException($e = new FatalThrowableError($e));

         $response = $this->renderException($request, $e);
     }

     $this->app['events']->dispatch(
         new Events\RequestHandled($request, $response)
     );

     return $response;
    }  
```

**handle()** 方法接收一个`HTTP`请求, 并最终生成一个`HTTP`响应.

继续深入到处理 `HTTP` 请求的方法 **$this->sendRequestThroughRouter($request)** 内部。

- `$this->sendRequestThroughRouter($request)`

```
    /**
     * Send the given request through the middleware / router.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);

        Facade::clearResolvedInstance('request');

        $this->bootstrap();

        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
```

将发现这段代码没有一行废话，它完成了大量的逻辑处理:

- 首先, 将`$request`实例注册到`APP`容器供后续使用
- 之后, 清除之前`$request`实例缓存
- 然后, 启动「引导程序」
- 最后, 发送请求至路由

###### 启动 「引导程序」

- 下面看看:`$this->bootstrap();`

在 `$this->bootstrap();` 方法内部有实际调用「引导程序」，而 `bootstrap()` 实际调用的是 `APP` 容器的 `bootstrapWith()`,来看看

```
    /**
     * Bootstrap the application for HTTP requests.
     *
     * @return void
     */
    public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }
    
    /**
     * Get the bootstrap classes for the application.
     *
     * @return array
     */
    protected function bootstrappers()
    {
        return $this->bootstrappers;
    }
```

最终还是要看 `Illuminate\Foundation\Application` 的 `bootstrapWith()` 方法究竟如何来启动这些引导程序的。

> 这里 按 `CTRL + 鼠标左键`追踪不到, 具体位置如下

- `\vendor\laravel\framework\src\Illuminate\Foundation\Application.php`

```
   /**
    * Run the given array of bootstrap classes.
    *
    * @param  array  $bootstrappers
    * @return void
    */
   public function bootstrapWith(array $bootstrappers)
   {
       $this->hasBeenBootstrapped = true;

       foreach ($bootstrappers as $bootstrapper) {
           $this['events']->fire('bootstrapping: '.$bootstrapper, [$this]);

           $this->make($bootstrapper)->bootstrap($this);

           $this['events']->fire('bootstrapped: '.$bootstrapper, [$this]);
       }
   } 
```

> 这里可以看到在 `APP` 容器内, 惠先解析对应的 「引导程序」 (即实例化), 随后调用 「引导程序」
的`bootstrap()`完成 「引导程序」来看看其内部的启动原理.

###### 发送请求至路由

完成「引导程序」启动操作后，随机进入到请求处理阶段。

```
    protected function sendRequestThroughRouter($request)
    {
        ...

        // 发送请求至路由
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
```

在 「发送请求至路由」这行代码中，完成了：
- 1. 管道（pipeline）创建
- 2. 将 `$request` 传入管道
- 3. 对 `$request` 执行「中间件」处理
- 4. 实际的请求处理
四个不同的操作。

> 传递给路由是通过 `Pipeline`（管道）来传递的，但是`Pipeline`有一堵墙，在传递给路由之前所有请求都要经过，
这堵墙定义在`app\Http\Kernel.php`中的`$middleware`数组中，没错就是中间件，
默认有一个`CheckForMaintenanceMode`中间件，用来检测你的网站是否暂时关闭。
这是一个全局中间件，所有请求都要经过，你也可以添加自己的全局中间件。

**那么，究竟一个请求是如何被处理的呢？**

```
    /**
     * Get the route dispatcher callback.
     *
     * @return \Closure
     */
    protected function dispatchToRouter()
    {
        return function ($request) {
            $this->app->instance('request', $request);

            return $this->router->dispatch($request);
        };
    }
```

通过 「解析内核实例」章节，可知我们已经将 `Illuminate\Routing\Router` 对象赋值给 `$this->router` 属性

通过 `router` 实例的 `disptach()` 方法去执行 HTTP 请求，在它的内部会完成如下处理：

1. 查找对应的路由实例
2. 通过一个实例盏运行给定的路由
3. 运行在`routes/web.php`配置的匹配到的控制器或匿名函数
4. 返回响应结果

执行 `$route->run()` 的方法定义在 `Illuminate\Routing\Route` 类中，最终执行「在 `routes/web.php` 配置的匹配到的控制器或匿名函数」:

```
    /**
     * Run the route action and return the response.
     *
     * @return mixed
     */
    public function run()
    {
        $this->container = $this->container ?: new Container;

        try {
            if ($this->isControllerAction()) {
                return $this->runController();
            }

            return $this->runCallable();
        } catch (HttpResponseException $e) {
            return $e->getResponse();
        }
    }
```

这部分如果路由的实现是一个控制器，会完成控制器实例化并执行指定方法；如果是一个匿名函数则直接调用这个匿名函数。

其执行结果会通过 `Illuminate\Routing\Router::prepareResponse($request, $response)` 生一个响应实例并返回。

至此，`Laravel` 就完成了一个 `HTTP` 请求的请求处理。

### 发送响应

发送响应由 `Illuminate\Http\Response` 父类 `Symfony\Component\HttpFoundation\Response` 中的 `send()` 方法完成

```
    /**
     * Sends HTTP headers and content.
     *
     * @return $this
     */
    public function send()
    {
        $this->sendHeaders();  // 发送响应头部信息
        $this->sendContent();  // 发送报文主题

        if (function_exists('fastcgi_finish_request')) {
            fastcgi_finish_request();
        } elseif (!\in_array(PHP_SAPI, array('cli', 'phpdbg'), true)) {
            static::closeOutputBuffers(0, true);
        }

        return $this;
    }
```

### 终止程序

> 程序终止, 完成终止中间件的调用

- `Illuminate\Foundation\Http\Kernel.php`

```
    /**
     * Call the terminate method on any terminable middleware.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Http\Response  $response
     * @return void
     */
    public function terminate($request, $response)
    {
        $this->terminateMiddleware($request, $response);

        $this->app->terminate();
    }
    
    /**
     * Call the terminate method on any terminable middleware.
     * 终止中间件
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Http\Response  $response
     * @return void
     */
    protected function terminateMiddleware($request, $response)
    {
        $middlewares = $this->app->shouldSkipMiddleware() ? [] : array_merge(
            $this->gatherRouteMiddleware($request),
            $this->middleware
        );

        foreach ($middlewares as $middleware) {
            if (! is_string($middleware)) {
                continue;
            }

            list($name) = $this->parseMiddleware($middleware);

            $instance = $this->app->make($name);

            if (method_exists($instance, 'terminate')) {
                $instance->terminate($request, $response);
            }
        }
    }
```

### 总结

![Laravel生命周期](https://caoxl.com/images/laravel-life.png)

> 在 「创建 Laravel 应用实例」时不仅会注册项目基础服务、注册项目服务提供者别名、注册目录路径等在内的一系列注册工作；
还会绑定 HTTP 内核及 Console 内核到 APP 容器， 同时在 HTTP 内核里配置中间件和引导程序。
> 
> 进入 「接收请求并响应」里，会依据运行环境从 `APP` 容器 解析出 `HTTP` 内核或 `Console` 内核。
如果是 `HTTP` 内核，还将把「中间件」及「引导程序」注册到 `APP` 容器。
> 
> 所有初始化工作完成后便进入「处理 `HTTP` 请求」阶段。
>
> 一个 `Http` 请求实例会被注册到 `APP` 容器，通过启动「引导程序」来设置环境变量、加载配置文件等等系统环境配置;
> 
> 随后请求被分发到匹配的路由，在路由中执行「中间件」以过滤不满足校验规则的请求，
只有通过「中间件」处理的请求才最终处理实际的控制器或匿名函数生成响应结果。
> 
> 最后发送响应给用户，清理项目中的中间件，完成一个 「请求」 - 「响应」 的生命周期，之后我们的 `Web` 服务器将等待下一轮用户请求。


# `Lumen` 生命周期

> `Lumen` 被称为简洁版 `Laravel`, 但是还是有很多差异的!

## 程序入口 `public/index.php`

- 加载`bootstrap/app.php`

```
    $app = require __DIR__.'/../bootstrap/app.php';
```

- 执行容器的`run()`

```
    $app->run();
```

## `bootstrap/app.php`中的操作

```php
    <?php
    
    require_once __DIR__.'/../vendor/autoload.php';
    
    try {
        (new Dotenv\Dotenv(__DIR__.'/../'))->load();
    } catch (Dotenv\Exception\InvalidPathException $e) {
        //
    }
    
    $app = new Laravel\Lumen\Application(
        realpath(__DIR__.'/../')
    );
    
    $app->withFacades();
    
    $app->withEloquent();
    
    $app->singleton(
        Illuminate\Contracts\Debug\ExceptionHandler::class,
        App\Exceptions\Handler::class
    );
    
    $app->singleton(
        Illuminate\Contracts\Console\Kernel::class,
        App\Console\Kernel::class
    );
    
    $app->middleware([
       App\Http\Middleware\ExampleMiddleware::class
    ]);
    
    $app->routeMiddleware([
        'auth' => App\Http\Middleware\Authenticate::class,
    ]);
    
    $app->register(App\Providers\AppServiceProvider::class);
    $app->register(App\Providers\AuthServiceProvider::class);
    $app->register(App\Providers\EventServiceProvider::class);
    
    $app->router->group([
        'namespace' => 'App\Http\Controllers',
    ], function ($router) {
        require __DIR__.'/../routes/web.php';
    });
    
    return $app;
```

**说明:**

- `require_once __DIR__.'/../vendor/autoload.php;`

> 加载`vendor`目录自动加载文件; 这是`composer`默认自动加载文件

- `()new Dotnev\Dotenv(__DIR__.'/../))->load();`

> 加载`.env`配置; `Lumen`采用`.env`接管配置, 此处进行配置加载

- `$app = new Laravel\Lumen\Application();`

> 生成容器`$app`; 其实`Laravel/Lumen`的核心就是容器, 再看`Application`源码也是继承了`Container`

- `$app->withFacades();`

> 启用`Facades`特性; `Lumen`默认是禁用了`Facades`特性, 取消注释则启用.

- `$app->withEloquent();`

> 启用`Eloquent`特性; `Lumen`默认禁用了`Eloquent`的`ORM`支持, **作者说是给用户自行选择不同`ORM`的权利

- `$app->configure('app');`

> 加载配置文件; 如果有自定义的配置文件, 则需要在此进行加载.

- `$app->singleton();`

> 注册容器绑定; 看过源码`singleton`和`bind`的区别就是前者内部调用了`bind`, 且第三个参数设为`true`, 意思是共享、独一的对象，也就是单例模式.

- `$app->middleware();`

> 注册中间件; 根据需要添加或去除注释启用

- `$app->register(App\Providers\LogServiceProvider::class);`

> 注册`ServiceProvider`; `Laravel`/`Lumen`的很大一部分特性是依赖`ServiceProvider`实现的

- `$app->group();`

> 加载路由文件; 这里可以再跟下`group()`的实现, 在`vendor/laravel/lumen-framework/src/Routing/Router.php`

- `return $app;`

> 返回容器`$app`

![Lumen生命周期](https://caoxl.com/images/lumen-life.png)

## `Laravel/Lumen` 容器

![Laravel/Lumen容器](https://caoxl.com/images/laravel-lumen-container.png)

### `Facades`模式

> 中文译名: 外观模式、门面模式
> 主要特性: 屏蔽内部实现, 开发接口供外部使用, 提高模块抽离

`Laravel`的`Facades`也是在框架级提供抽象调用, 从而是开发者无需关心内容实现.

**Facades的理解:**

1. 别名, 首先通过`class_alias`将`DB`、`Log`、`Cache`等关键词映射为不同的`Facades`类

```
    class_alias('Illuminate\Support\Facades\Cache', 'Cache');
```

2. 不同`Facades`子类返回容器内注册服务的关键词`db`、`log`、`cache`等

> `vendor/illuminate/support/Facades/`

```php
    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor()
        {
            return 'cache';
        }
    }
```

3. 通过绑定函数,向容器注册或绑定`ServiceProvider`

> `/vendor/laravel/lumen-framework/src/Application.php`

```
    protected function registerCacheBindings()
    {
        $this->singleton('cache', function () {
            return $this->loadComponent('cache', 'Illuminate\Cache\CacheServiceProvider');
        });
        $this->singleton('cache.store', function () {
            return $this->loadComponent('cache', 'Illuminate\Cache\CacheServiceProvider', 'cache.store');
        });
    }
```

`Facades`主要依赖`Service Container`和`Service Provider`实现

> 推荐一篇文章对容器介绍很全面: [laravel 学习笔记 —— 神奇的服务容器](https://www.insp.top/article/learn-laravel-container)

### `Service Container`

其实就是`$app`, 它是全局的一个容器，是`Lumen`的核心。

> 容器(`Service Container`)实现: `vendor/laravel/lumen-framework/src/Application.php`

```
    class Application extends Container
    {
        use Concerns\RoutesRequests,
            Concerns\RegistersExceptionHandlers;
        public function register($provider) {}
        public function make($abstract, array $parameters = []) {}
        public function configure($name) {}
        public function withFacades($aliases = true, $userAliases = []) {}
        public function withAliases($userAliases = []) {}
        public function withEloquent() {}
        protected function registerCacheBindings() {}
        protected function registerDatabaseBindings() {}
        protected function registerEncrypterBindings() {}
        protected function registerLogBindings() {}
        ...
        // 各种`Service Provider`的绑定
        ...
    }
```

### `Service Provider`

向容器提供不同的服务, 通过`register`函数向容器中注入类实例, 从而实现`DI`(依赖注入), 是一种`IOC`的实现方式, 不同服务的`ServiceProvider`分布在不同额服务子目录下

不同服务的`ServiceProvider`分布在不同的服务子目录下:

- `cache: vendor/illuminate/cache/CacheServiceProviders.php`

- `db: vendor/illuminate/database/DatabaseServiceProvider.php`

- `queue: vendor/illminate/queue/QueueServiceProvider.php`

- 不同的`ServiceProvider`都继承`ServiceProvider`

- 重写`register()`方法

- `$defer`变量表征是否延迟加载

- `providers()`方法返回该`ServiceProvider`可向容器提供的所有服务列表

# 参考

- [深度挖掘 Laravel 生命周期](https://segmentfault.com/a/1190000014598466)
- [Laravel 的生命周期](https://www.jianshu.com/p/08b810b720d9)-- 该`Laravel`版本较低
- [laravel 学习笔记 —— 神奇的服务容器](https://www.insp.top/article/learn-laravel-container)