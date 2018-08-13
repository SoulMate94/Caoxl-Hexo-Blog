---
title: Laravel 执行流程
date: 2018-02-23 10:47:55
categories: Laravel
tags: [Laravel]
---

分析Laravel执行流程.

<!--more-->

# 一、自动加载

## 定位

> - 从 `public/index.php` 定位到 `bootstrap/autoload.php`
> - 从 ` bootstrap/autoload.php` 定位到 `_vendor/autoload.php`
> - 从 `vendor/autoload.php` 定位到 `__DIR__ . '/composer' . '/autoload_real.php';`

定位完毕, 你会看到这样的代码:

```
    return ComposerAutoloaderInitda859f5bf8329e41e463c784e73d3cdf::getLoader();
```

**ComposerAutoloaderInitda859f5bf8329e41e463c784e73d3cdf** 简称**本类**

那我们就从 `getLoader()` 方法入手。

## `getLoader()`

文件位于： `__DIR__ . '/composer' . '/autoload_real.php'`;

逻辑顺序：

一、如果静态变量 `$loader` 不为空则返回 `$loader`,

```
    if (null !== self::$loader) {
        return self::$loader;
    }
```

二、注册一个自动加载程序，加载程序为本类的 `loadClassLoader()` 方法,

```
    spl_autoload_register(array('ComposerAutoloaderInitda859f5bf8329e41e463c784e73d3cdf', 'loadClassLoader'), true, true);
    
    self::$loader = $loader = new \Composer\Autoload\ClassLoader();
```

**`[loadClassLoader 方法逻辑]`**

```
    public static function loadClassLoader($class)
    {
        if ('Composer\Autoload\ClassLoader' === $class) {
            require __DIR__ . '/ClassLoader.php';
        }
    }
```

静态方法，含有一个 `$class` 参数，判断如果 `$class` 等于 `Composer\Autoload\ClassLoader`，则载入当前目录下 的 `ClassLoader.php` 文件，实际上是在为这句代码工作：

```
    self::$loader = $loader = new \Composer\Autoload\ClassLoader();
```

三、`$loader` 得到 `ClassLoader` 类（`\Composer\Autoload\ClassLoader`）的一个实例，卸载自动加载程序 `loadClassLoader`，

```
    self::$loader = $loader = new \Composer\Autoload\ClassLoader();
    
    spl_autoload_unregister(array('ComposerAutoloaderInit022a3829915a6a8f070076dbcb34819c', 'loadClassLoader'));
```

四、载入路径信息，设置路径信息，

五、载入一些 `autoload_x.php` 形式的文件，

分别有:

- `autoload_namespaces.php`
- `autoload_psr4.php`
- `autoload_classmap.php`

并进行各自的循环 set 操作，如 

```
    $loader->set($namespace, $path);
    
    $loader->setPsr4($namespace, $path);
    
    $loader->addClassMap($classMap);
```

> 【set 函数】2个参数，`一个前缀`，`一个路径`。如果前缀非真，将 `paths` 转为数组类型赋值给类成员变量` fallbackDirsPsr0`，如果前缀为真，则将路径赋值给 `$this->prefixesPsr0[$prefix[0]][$prefix]` ，这个写法的意思等同于字母索引，比如 `phpDocumentor` ，则数组就图所示：
 
 ![](https://dn-phphub.qbox.me/uploads/images/201506/18/1795/JTaL9Qxyp9.png)
 
 六、执行一个 `$loader->register(true);`,
 
**[register 方法逻辑]**

```
    $loader->register(true);
```

一个布尔值参数，将传给 `spl_autoload_register` 第三个参数中。 而自动加载程序为：`array($this, 'loadClass')`，也就是本类的 `loadClass()` 方法。

**[loadClass 方法逻辑]**

```
        public function loadClass($class)
        {
            if ($file = $this->findFile($class)) {
                includeFile($file);
    
                return true;
            }
        }
```

一个 `$class` 参数，用了 `findFile()` 方法判断文件是否存在，存在则调用函数 `includeFile()` 载入文件

```
    function includeFile($file)
    {
        include $file;
    }
```

七、还载入了一个`autoload_files.php`，而里面也是一组文件数组，貌似预加载一些函数库文件吧，没有继续深入这里了。

- `verdor/composer/autoload_files.php`

```
    if ($useStaticLoader) {
        $includeFiles = Composer\Autoload\ComposerStaticInit022a3829915a6a8f070076dbcb34819c::$files;
    } else {
        $includeFiles = require __DIR__ . '/autoload_files.php';
    }
```

八、最后返回一个 `$loader` 变量，也就是 `ClassLoader` 类的实例。

```
    foreach ($includeFiles as $fileIdentifier => $file) {
        composerRequire022a3829915a6a8f070076dbcb34819c($fileIdentifier, $file);
    }

    return $loader;
```

```
    function composerRequire022a3829915a6a8f070076dbcb34819c($fileIdentifier, $file)
    {
        if (empty($GLOBALS['__composer_autoload_files'][$fileIdentifier])) {
            require $file;
    
            $GLOBALS['__composer_autoload_files'][$fileIdentifier] = true;
        }
    }
```

好了，现在看看 `$loader` 这个实例到此拥有些什么？部分截图所示：

![$loader](https://dn-phphub.qbox.me/uploads/images/201506/18/1795/yqHs8C9jDt.png)

可以看出类属性包含了具有字母索引的一些命名空间，文件路径等信息。这和刚才载入那几个文件进行 `set` 操作有关，想起来了吗？

到此 `getLoader()` 方法逻辑结束。

## 总结

实现**自动化**的关键代码是 `vendor/autoload.php` 的 `::getLoader()` 静态方法， 利用此方法内部的 `$loader->register(true)`; 方法**注册自动化载入方法**，这样，当 `new` 对象的时候自动触发 `loadClass()` 了，而上面提到的 `set` 一些路径信息，正是自动化的必备条.

如有兴趣可以自行查看 `vendor/composer/ClassLoader.php` 的 `loadClass()` 方法代码细节。

> 上面如果没懂的，请打开文件代码，跟着慢慢走，慢慢看，一定能懂。

再返回到 `vendor/autoload.php`，再把 `return $loader` 返回到上一层。 即 `bootstrap/autoload.php`， 

这行的代码 `require __DIR__.'/../vendor/autoload.php`'; 

我们 `var_dump()` 下 `require` 的返回值，和刚才 `$loader` 的部分截图完全一致。 其实有从 `aotuload_real.php` 文件开始，我尝试过删除 `return`，也没有任何报错，不知道这里的 `return` 意义为何

再看 `index.php` 定位到了 `bootstrap/app.php` 打开就看到第一个

```
    $app = new Illuminate\Foundation\Application(
        realpath(__DIR__.'/../')
    );
```

下面来详细介绍:

# Container 类

## 定位

- 来到 `bootstrap/app.php`

发现一段英文注释:

```
    /*
    |--------------------------------------------------------------------------
    | Create The Application
    |--------------------------------------------------------------------------
    |
    | The first thing we will do is create a new Laravel application instance
    | which serves as the "glue" for all the components of Laravel, and is
    | the IoC container for the system binding all of the various parts.
    |
    */
```

翻译过来:

```
    / *
    | --------------------------------------------------------------------------
    |创建应用程序
    | --------------------------------------------------------------------------
    |
    |我们首先要做的是创建一个新的 laravel 应用实例
    |作为“胶水”用于 laravel 的所有组件，并
    |为系统结合各种零件的 IOC 容器。
    |
    * /
```

> 换句话说，把这个**对象实例**作为一个可依靠稳固的**地基**，来存放各种**“建筑物（组件）”**，So **地基= IOC 容器？** 好复杂有木有，不管它了，读代码去吧，读代码就像聊天，得用心倾听别人的心声。

紧接着 `new` 了一个对象。 

```
    $app = new Illuminate\Foundation\Application(
        realpath(__DIR__.'/../')
    );
```

`new Illuminate\Foundation\Application()` 构造函数做了什么？

`Application` 类还传入了当前 laravel 根目录的地址这样一个参数。如 `“D:\webSite\Laravel”` 

```
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

命名空间 `Illuminate` 的具体文件路径位于 `Vendor/laravel/framework/src/Illuminate`。

打开文件: `Vendor/laravel/framework/src/Illuminate/Application.php`

到这里: `class Application extends Container implements ApplicationContract, HttpKernelInterface`

继承 `Container` 类，对应上面的use：`use Illuminate\Container\Container`; 并且要实现2个接口: `ApplicationContract`，`HttpKernelInterface`

 咱们先看看这个 `Container` 类到底是什么。 怎么找具体文件上面已经说过了，下面就直接略过，直接打开 `Container.php`。

## `Container` 类

- `Vendor/laravel/framework/src/Illuminate/Container/Container.php`

`The ArrayAccess interface` 数据访问接口，PHP5的一个新接口，**接口提供访问对象数组**。

```
    class Container implements ArrayAccess, ContainerContract
```

需要实现四个方法:

- `abstract public boolean offsetExists ( mixed $offset )`
- `abstract public mixed offsetGet ( mixed $offset )`
- `abstract public void offsetSet ( mixed $offset , mixed $value )`
- `abstract public void offsetUnset ( mixed $offset )`

先不用懂这4个方法是什么意思，看看 `Container` 类怎么实现的就懂了。

### `offsetExists`

```
    public function offsetExists($key)
    {
        return $this->bound($key);
    }
```

```
    public function bound($abstract)
    {
        return isset($this->bindings[$abstract]) ||
               isset($this->instances[$abstract]) ||
               $this->isAlias($abstract);
    }
```

当外部使用 `isset()` 函数时触发此方法，而方法内部则也是使用一个 `isset` 来判断返回一个 `bool` 值。这个 `bindings` 先不管，我们主要知道这个方法做了什么事情。

### `offsetGet`

```
    public function offsetGet($key)
    {
        return $this->make($key);
    }
```
当外部尝试获取 `$key` 的时候，如 `echo` ,`print_r`,`var_dump`，赋值等。 就会触发此方法，而方法内部也是直接一个 `reutrn`，简单！

### `offsetSet`

```
        public function offsetSet($key, $value)
        {
            $this->bind($key, $value instanceof Closure ? $value : function () use ($value) {
                return $value;
            });
        }
```

当外部尝试赋值的时候，则触发此方法。 此方法内部先进行一个 `instanceof` 判断 如果 `value` 参数不是一个闭包，则将 `$value` 封装成一个闭包，而闭包函数的内部直接一个 `return`。

### `offsetUnset`

```
    public function offsetUnset($key)
    {
        unset($this->bindings[$key], $this->instances[$key], $this->resolved[$key]);
    }
```

当外部尝试使用 `unset` 函数的时候触发此方法， 方法内部是一个 `unset` 函数，那些变量看不懂，先不管。

这就是 `ArrayAccess`，一个类实现它的4个方法，在**不同操作时触发4个方法**，知道这些就够了，具体调用的示例这里我没有举出来，PHP 官网有，请自行翻阅,锻炼下动手能力。

好吧继续往下看;

- `ReflectionMethod ` 针对类方法
- `ReflectionFunction` 针对普通函数
- `ReflectionParameter` 针对函数的参数

具体代码演示如下：

```
    /* ReflectionMethod 示例
    $a = new ReflectionMethod('index\index','a');
    var_dump($a->isAbstract()); //false
    */
    
    /* ReflectionParameter 示例
    $a = new ReflectionParameter('index\b',2);
    var_dump($a->getName()); //string(1) "d"
    */
    
    /* ReflectionFunction 示例
    $a = new ReflectionFunction('index\b');
    var_dump($a->getParameters());
    array(3) {
      [0]=>
      &object(ReflectionParameter)#2 (1) {
        ["name"]=>
        string(1) "b"
      }
      [1]=>
      &object(ReflectionParameter)#3 (1) {
        ["name"]=>
        string(1) "c"
      }
      [2]=>
      &object(ReflectionParameter)#4 (1) {
        ["name"]=>
        string(1) "d"
      }
    }
    */
```

然后看到 `Container` 类还实现这个接口 `ContainerContract`。

对应 `use Illuminate\Contracts\Container\Container as ContainerContract`;

提示：`Contracts` 翻译 *“合同、契约”*。

接口里面的方法有兴趣可自行查看，我们主要看 `Container` 的实现这些方法的细节。

> 最后我们简述下 Contracts 作为本章结束语： 我的理解是一系列组件服务的接口集合

# 探索 Application 构造函数

## 定位

- OK，从入口地址 `public/index.php` 看到如下代码：

```
    /*
    |--------------------------------------------------------------------------
    | Turn On The Lights
    |--------------------------------------------------------------------------
    |
    | We need to illuminate PHP development, so let us turn on the lights.
    | This bootstraps the framework and gets it ready for use, then it
    | will load up this application so that we can run it and send
    | the responses back to the browser and delight our users.
    |
    */
    
    $app = require_once __DIR__.'/../bootstrap/app.php';
```

**翻译:**

![](https://dn-phphub.qbox.me/uploads/images/201509/25/1795/R8XmO21d3w.png)

注释很有趣，翻译凑合看吧。 则现在我们打开 `bootstarp/app.php` 文件，因为这是 `$app` 这个玩意儿的出生地。

```
    /*
    |--------------------------------------------------------------------------
    | Create The Application
    |--------------------------------------------------------------------------
    |
    | The first thing we will do is create a new Laravel application instance
    | which serves as the "glue" for all the components of Laravel, and is
    | the IoC container for the system binding all of the various parts.
    |
    */
    
    $app = new Illuminate\Foundation\Application(
        realpath(__DIR__.'/../')
    );
```

**翻译:**

![](https://dn-phphub.qbox.me/uploads/images/201509/25/1795/YyxYu1Zwgt.png)

## `Application.php`

咱们就从在这里摸索一下构造函数里面发生了什么，打开 `Application.php` 文件。

打开的 `Application.php` 是位于 `vendor/laravel/framework/src/Illuminate/Foundation/Application.php`，而命名空间是 `Illuminate\Foundation`

`Application`类的构造函数如下：

```
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

因为代码量真的很多，如果每一点都要说，不仅会看得云里雾里，还不一定能理解，所以，我这里以总结的方式概述，特别值得学习的地方再单独提出来说。

### 一、`registerBaseBindings()`

> 注册一些基本的绑定到容器中。

简单点说，此方法内部进行`3`次赋值，赋值后的变量及变量内容形式如下：

> - ` static::setInstance($this);`
> - `$this->instance('app', $this);`
> - `$this->instance(Container::class, $this);`

**借鉴:**

```
    Container::$instance = $this
    
    $this->instances['app'] = $this
    
    $this->instances['Illuminate\Container\Container'] = $this
```

变量名具体含义:

`$this`，也就是 `Application` 类。

static::setInstance，之前说过 `Application` 是 `Container` 的子类，而 `$instance` 静态变量是在 `Container` 类中已经定义好的,：

- `Application.php`

```
    class Application extends Container implements ApplicationContract, HttpKernelInterface
    {
    }
```

- `Container.php`

```
    class Container implements ArrayAccess, ContainerContract
    {
        protected static $instance;
    }
```

`$this->instances`，也是在` Container` 中定义的，含义为存放容器的共享实例,

```
    /**
     * The container's shared instances.
     *
     * @var array
     */
    protected $instances = [];
```

你可以在 `registerBaseBindings` 方法的最后面打印如下3个变量进行检测，得到的都是 `application object`，

```
    protected function registerBaseBindings()
    {
        static::setInstance($this);

        $this->instance('app', $this);

        $this->instance(Container::class, $this);

        $this->instance(PackageManifest::class, new PackageManifest(
            new Filesystem, $this->basePath(), $this->getCachedPackagesPath()
        ));

        print_r(static::$instance);die;
        print_r($this->instances);die;
    }
```

到此，所谓的基本绑定结束，还是云里雾里的，英文不好只能看代码了，反正你记住父类的2个成员属性已经得到了 `application` 对象。

### 二、`registerBaseServiceProviders()`

> 注册所有的基础服务提供商。

好吧，第二章提过的 `ioc 容器=地基`，开始买材料准备施工，找几个最基础的供应商商来进行合作，搞水泥的啊，砌砖的啊，以后有更多的需求，根据自己的需求在去找供应商谈。`Laravel` 现在注册了`3`个提供商，**一个事件**，**一个是路由**, **一个日志**。

```
    protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));

        $this->register(new LogServiceProvider($this));

        $this->register(new RoutingServiceProvider($this));
    }
```

> 我们先意淫一下大概的意思，找到了供应商，准备合作签合同( `register` )，先和搞水泥的签( `event` )，在和砌砖的签( `routing` )，和谁签？I'm the boss( `$this` )。 既然我是老板，合同条款肯定得看清楚了，咱们去看看合同先( `register`方法 ) 嗯！合同说的很清楚，首先确定我和供应商是否签过合同了，签过了( `getProvider()` 来判断)就滚蛋，浪费时间；虽然我是老板，但不是法人，你打个电话叫他过来，名字叫狗蛋（`resolveProvider` 方法，如果 `$provider` 为 `string` 类型，则根据提供的类名帮供应商实例化并 `return`），如:

```
    public function resolveProvider($provider)
    {
        return new $provider($this);
    }
```

> OK，差不多，狗蛋把字一签( `$provider->register()` )，供应商算是正式入驻施工团队了，当然了，合同还说明以后要是有其他要改的地方，直接填一份声明即可，`$options` 是 `register`方法 的第二个参数。

```
    public function register($provider, $options = [], $force = false)
    {
        if (($registered = $this->getProvider($provider)) && ! $force) {
            return $registered;
        }

        // If the given "provider" is a string, we will resolve it, passing in the
        // application instance automatically for the developer. This is simply
        // a more convenient way of specifying your service provider classes.
        if (is_string($provider)) {
            $provider = $this->resolveProvider($provider);
        }

        if (method_exists($provider, 'register')) {
            $provider->register();
        }

        $this->markAsRegistered($provider);

        // If the application has already booted, we will call this boot method on
        // the provider class so it has an opportunity to do its boot logic and
        // will be ready for any usage by this developer's application logic.
        if ($this->booted) {
            $this->bootProvider($provider);
        }

        return $provider;
    }
```

> 既然签了合同，就要如公司档案，狗蛋屁颠屁颠的跑去档案室了（`$this->markAsRegistered($provider)` ）标记为已注册；好，大功告成( `return $provider` )。


### 三、`registerCoreContainerAliases()`

> 注册核心容器的别名。

嗯，这个简单的多，还有啥好说的呢，定义容器里面一些核心类的别名，有兴趣直接去看这个方法就行。

```
    public function registerCoreContainerAliases()
    {
        foreach ([
            'app'                  => [\Illuminate\Foundation\Application::class, \Illuminate\Contracts\Container\Container::class, \Illuminate\Contracts\Foundation\Application::class,  \Psr\Container\ContainerInterface::class],
            'auth'                 => [\Illuminate\Auth\AuthManager::class, \Illuminate\Contracts\Auth\Factory::class],
            'auth.driver'          => [\Illuminate\Contracts\Auth\Guard::class],
            'blade.compiler'       => [\Illuminate\View\Compilers\BladeCompiler::class],
            'cache'                => [\Illuminate\Cache\CacheManager::class, \Illuminate\Contracts\Cache\Factory::class],
            'cache.store'          => [\Illuminate\Cache\Repository::class, \Illuminate\Contracts\Cache\Repository::class],
            'config'               => [\Illuminate\Config\Repository::class, \Illuminate\Contracts\Config\Repository::class],
            'cookie'               => [\Illuminate\Cookie\CookieJar::class, \Illuminate\Contracts\Cookie\Factory::class, \Illuminate\Contracts\Cookie\QueueingFactory::class],
            'encrypter'            => [\Illuminate\Encryption\Encrypter::class, \Illuminate\Contracts\Encryption\Encrypter::class],
            'db'                   => [\Illuminate\Database\DatabaseManager::class],
            'db.connection'        => [\Illuminate\Database\Connection::class, \Illuminate\Database\ConnectionInterface::class],
            'events'               => [\Illuminate\Events\Dispatcher::class, \Illuminate\Contracts\Events\Dispatcher::class],
            'files'                => [\Illuminate\Filesystem\Filesystem::class],
            'filesystem'           => [\Illuminate\Filesystem\FilesystemManager::class, \Illuminate\Contracts\Filesystem\Factory::class],
            'filesystem.disk'      => [\Illuminate\Contracts\Filesystem\Filesystem::class],
            'filesystem.cloud'     => [\Illuminate\Contracts\Filesystem\Cloud::class],
            'hash'                 => [\Illuminate\Contracts\Hashing\Hasher::class],
            'translator'           => [\Illuminate\Translation\Translator::class, \Illuminate\Contracts\Translation\Translator::class],
            'log'                  => [\Illuminate\Log\Writer::class, \Illuminate\Contracts\Logging\Log::class, \Psr\Log\LoggerInterface::class],
            'mailer'               => [\Illuminate\Mail\Mailer::class, \Illuminate\Contracts\Mail\Mailer::class, \Illuminate\Contracts\Mail\MailQueue::class],
            'auth.password'        => [\Illuminate\Auth\Passwords\PasswordBrokerManager::class, \Illuminate\Contracts\Auth\PasswordBrokerFactory::class],
            'auth.password.broker' => [\Illuminate\Auth\Passwords\PasswordBroker::class, \Illuminate\Contracts\Auth\PasswordBroker::class],
            'queue'                => [\Illuminate\Queue\QueueManager::class, \Illuminate\Contracts\Queue\Factory::class, \Illuminate\Contracts\Queue\Monitor::class],
            'queue.connection'     => [\Illuminate\Contracts\Queue\Queue::class],
            'queue.failer'         => [\Illuminate\Queue\Failed\FailedJobProviderInterface::class],
            'redirect'             => [\Illuminate\Routing\Redirector::class],
            'redis'                => [\Illuminate\Redis\RedisManager::class, \Illuminate\Contracts\Redis\Factory::class],
            'request'              => [\Illuminate\Http\Request::class, \Symfony\Component\HttpFoundation\Request::class],
            'router'               => [\Illuminate\Routing\Router::class, \Illuminate\Contracts\Routing\Registrar::class, \Illuminate\Contracts\Routing\BindingRegistrar::class],
            'session'              => [\Illuminate\Session\SessionManager::class],
            'session.store'        => [\Illuminate\Session\Store::class, \Illuminate\Contracts\Session\Session::class],
            'url'                  => [\Illuminate\Routing\UrlGenerator::class, \Illuminate\Contracts\Routing\UrlGenerator::class],
            'validator'            => [\Illuminate\Validation\Factory::class, \Illuminate\Contracts\Validation\Factory::class],
            'view'                 => [\Illuminate\View\Factory::class, \Illuminate\Contracts\View\Factory::class],
        ] as $key => $aliases) {
            foreach ($aliases as $alias) {
                $this->alias($key, $alias);
            }
        }
    }
```

当然了最后是存放在 `$aliases` 这个数组里面哟，在 `container` 定义的成员属性。

### 四、`setBasePath()`

> 设置基本路径

```
    public function setBasePath($basePath)
    {
        $this->basePath = rtrim($basePath, '\/');

        $this->bindPathsInContainer();

        return $this;
    }
```

```
    protected function bindPathsInContainer()
    {
        $this->instance('path', $this->path());
        $this->instance('path.base', $this->basePath());
        $this->instance('path.lang', $this->langPath());
        $this->instance('path.config', $this->configPath());
        $this->instance('path.public', $this->publicPath());
        $this->instance('path.storage', $this->storagePath());
        $this->instance('path.database', $this->databasePath());
        $this->instance('path.resources', $this->resourcePath());
        $this->instance('path.bootstrap', $this->bootstrapPath());
    }
```

```
    protected $basePath;

    public function path($path = '')
    {
        return $this->basePath.DIRECTORY_SEPARATOR.'app'.($path ? DIRECTORY_SEPARATOR.$path : $path);
    }
```

这个更简单了，这就是前面说 `$app` 出生地的地方，传了一个路径参数,

```
    $app = new Illuminate\Foundation\Application(
        realpath(__DIR__.'/../')
    );
```

至此，`$app` 终于生出来了，绑定了 `application` 对象，和`3`个供应商签了合同，给一些`核心类起了别名`，配置了 `laravel` 根目录地址

# 认识 `Bind`

第三章的探索相信有很多学友只是知道一些基本流程，并未真正意义上知道其所以然，不过学习还需要循序渐进，先了解，在深入，最后总结，本章，咱们将要探秘 `bind()` 方法。

- `Bind`方法位于`Container.php`

```
    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }
```

Bind 第一次被使用是在 EventServiceProvider 类的 register() 方法里

- `vendor/laravel/framework/src/Illuminate/Events/EventServiceProvider.php`

第三章说到注册3个基本的服务提供商，分别是事件( event )和路由( route )还有日志(Log),都分别执行了各自的 $provider->register()。

首先看第一次执行 `bind()` 的代码上下文：

```
    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton('events', function ($app) {
            return (new Dispatcher($app))->setQueueResolver(function () use ($app) {
                return $app->make(QueueFactoryContract::class);
            });
        });
    }
```

`$this->app` 怎么来的？这个成员属性是在抽象类 `ServiceProvider` 中定义好的，并在其构造函数中进行赋值：

```
    abstract class ServiceProvider
    {
        /**
         * The application instance.
         *
         * @var \Illuminate\Contracts\Foundation\Application
         */
        protected $app;
        
        /**
         * Create a new service provider instance.
         *
         * @param  \Illuminate\Contracts\Foundation\Application  $app
         * @return void
         */
        public function __construct($app)
        {
            $this->app = $app;
        }
    }
```

大家可以查阅，所有的服务提供商都继承于这个类，而在 `application` 类中的 `registerBaseServiceProviders()` 传入了这个参数 `$this`：

```
    $this->register(new EventServiceProvider($this));
```

接下来我们看见的是 `singleton` 并非 `bind` 方法，而 `singleton` 意为单例，在方法内部调用如下：

- vendor/laravel/framework/src/Illmiunate/Container/Container.php

```
     /**
      * Register a shared binding in the container.
      *
      * @param  string|array  $abstract
      * @param  \Closure|string|null  $concrete
      * @return void
      */
     public function singleton($abstract, $concrete = null)
     {
         $this->bind($abstract, $concrete, true);
     }
```

只不过是把 `bind` 的第三个参数设置为共享状态。

好，结合现在的情况，在分析下参数，给了一个 `events` 字符串，还有一个闭包函数，闭包需要传入一个参数 `$app` ，这个参数无非就是 `application` 的实例。

好，把这个来龙去脉弄清楚了，我们才能更清晰的分析 `bind` 内部逻辑。

Bind 开头进行了一个数组判断，为了大家看的清楚，我简单的举了个例子：

![](https://dn-phphub.qbox.me/uploads/images/201510/03/1795/7pnmO6Voq1.png)

`extractAlias` 方法只是返回了这样一个数组：

![](https://dn-phphub.qbox.me/uploads/images/201510/03/1795/drL6WHCQ7W.png)

很简单吧，提取后在重组别名：

![](https://dn-phphub.qbox.me/uploads/images/201510/03/1795/sx1BrfDkn7.png)

代码带来的功能就是这样，具体的意义，我把注释翻译，大家尽量去理解：

```
    // If no concrete type was given, we will simply set the concrete type to the
    // abstract type. After that, the concrete type to be registered as shared
    // without being forced to state their classes in both of the parameters.
    
    // 如果没有具体类型($concrete), 我们将简单的将具体类型设置为抽象类型 (也就是$concrete = $abstract)
    // 这将允许具体的类型被注册为共享, 而无需在两个参数($concrete, $abstract)中说明其类.
    
    // 删除旧的实例
    $this->dropStaleInstances($abstract);
    
```

- `dropStaleInstances`

```
    protected function dropStaleInstances($abstract)
    {
        unset($this->instances[$abstract], $this->aliases[$abstract]);
    }
```

删除以后，判断了如下：

```
     if (is_null($concrete)) {
         $concrete = $abstract;
     }
```

为了把这里弄懂，我们现在假设 `$concrete` 确实为 `null`，然后看往下是怎么发展的。

```
    // If the factory is not a Closure, it means it is just a class name which is
    // bound into this container to the abstract type and we will just wrap it
    // up inside its own Closure to give us more convenience when extending.
    
    if (! $concrete instanceof Closure) {
        $concrete = $this->getClosure($abstract, $concrete);
    }
```

如果 `$concrete` 不是一个`闭包`，就封装成一个`闭包`，刚才我们假设了为 `null`，并且 `$concrete` = `$abstract`，那么在看一下 `getClosure` 方法：

```
    protected function getClosure($abstract, $concrete)
    {
        return function ($container, $parameters = []) use ($abstract, $concrete) {
            if ($abstract == $concrete) {
                return $container->build($concrete);
            }

            return $container->make($concrete, $parameters);
        };
    }
```

我们以刚才的假设来虚拟场景，`$concrete = $abstract`,所以 `$method=build`，并且执行 `$container->build($concrete)` 方法，

接下来要认识一个新数组 `bindings`：

```
    $this->bindings[$abstract] = compact('concrete', 'shared');
```

`Compact` 创建一个由参数所带变量组成的数组。如果参数中存在数组，该数组中变量的值也会被获取，这里我们知道 `$conrete` 被封装成一个闭包了，而 `$shared` 是在 `singleton` 方法中传的 `true`，也就是`1`。所以现在`bindings` 数组形式如下：

![bindings 数组形式](https://dn-phphub.qbox.me/uploads/images/201510/03/1795/rOP6CGGP48.png)

这里有部分人可能会纠结，你刚才不是说 `$concrete` 为 `null`，怎么又变成了一个闭包对象；刚才仅仅只是用假设，来弄懂一个环节，所以正常来说这里就是这样。

```
    // If the abstract type was already resolved in this container we'll fire the
    // rebound listener so that any objects which have already gotten resolved
    // can have their copy of the object updated via the listener callbacks.
    
    // 如果$abstract 已经解析的话, 则反弹给监听者, 那么任何一个已经解析的对象 就能通过监听者的回调函数进行更新
    
    if ($this->resolved($abstract)) {
        $this->rebound($abstract);
    }
```

`Events` 肯定没有解析过，所以这里的 `rebound` 反弹暂时不能过多讲解。

到现在为止，闭包内的代码依然没有执行，而 `bind` 方法也到此为止了。

## 总结

`Bind` 方法不仅可以**智能的创建快捷方式**，还可以**封装闭包函数**，并**把所有的对应绑定都存入到 `bindings` 数组中**，如果一旦某个对象解析过了依然调用 `bind` 方法，那么就会 `rebound` 反弹，**通过监听者的回调函数更新对象**。

# 认识 `Make`

上一章了解了 `bind`，`bind` 封装了一个闭包到数组 `bindings`，那绑定完了总需要使用吧，就好像生产零件，先规定了怎么生产( `bind` )，然后再生产( `make` )所以 `make` 出来了，咱们一起看看，`make` 在容器里面起到什么作用。

还是老办法，先找出第一次调用 `make` 的地方，之前第三章探索 `application` 构造函数还是很有必要的，因为若干的第一次 `work`，以及认识这些关键的方法都包含在内，如果对以下所说的方法比较模糊，自行回顾第三章，并打开代码走一遍。

我们知道 `application` 构造函数里面调用了 `registerBaseServiceProviders` （注册基本的服务提供商），内部调用了 `register` 方法，`register` 方法内部又调用了 `markAsRegistered` （标记为已注册），咱们现在看 `markAsRegistered` 方法：

- `Application.php@markAsRegistered`

```
    protected function markAsRegistered($provider)
    {
        $this->serviceProviders[] = $provider;

        $this->loadedProviders[get_class($provider)] = true;
    }
```

```
    function get_class ($object = null) {}
```


## 正确执行 Make

> `vendor/laravel/framework/src/Illuminate/Container@offsetGet`

- `offsetGet`

```
    public function offsetGet($key)
    {
        return $this->make($key);
    }
```

- `make`

```
    public function make($abstract, array $parameters = [])
    {
        return $this->resolve($abstract, $parameters);
    }
```

- `resolve`

```
    protected function resolve($abstract, $parameters = [])
    {
    }
```

先是获取正确的别名 `getAlias`。

```
    $abstract = $this->getAlias($abstract);
```

接下来也是一个单例的判断，如果当前 `instances` 数组包含此类型，则直接 `return` 此单元。

```
    if (isset($this->instances[$abstract]) && ! $needsContextualBuild) {
        return $this->instances[$abstract];
    }
```

接着执行了一个新玩意 `getConcrete` 并将结果赋值给 `$concrete`，咱们在跳过去看看 `getConcrete` ，这个方法的大概意思是得到一个给定的抽象的具体类型。

```
    $concrete = $this->getConcrete($abstract);
```

- `getConcrete`

```
    protected function getConcrete($abstract)
         {
             if (! is_null($concrete = $this->getContextualConcrete($abstract))) {
                 return $concrete;
             }
     
             // If we don't have a registered resolver or concrete for the type, we'll just
             // assume each type is a concrete name and will attempt to resolve it as is
             // since the container should be able to resolve concretes automatically.
             if (isset($this->bindings[$abstract])) {
                 return $this->bindings[$abstract]['concrete'];
             }
     
             return $abstract;
         }
```

先是来了个看不懂的方法 `getContextualConcrete` 不明觉厉:

```
    protected function getContextualConcrete($abstract)
    {
        if (! is_null($binding = $this->findInContextualBindings($abstract))) {
            return $binding;
        }

        // Next we need to see if a contextual binding might be bound under an alias of the
        // given abstract type. So, we will need to check if any aliases exist with this
        // type and then spin through them and check for contextual bindings on these.
        if (empty($this->abstractAliases[$abstract])) {
            return;
        }

        foreach ($this->abstractAliases[$abstract] as $alias) {
            if (! is_null($binding = $this->findInContextualBindings($alias))) {
                return $binding;
            }
        }
    }
```

- `findInContextualBindings`

```
    protected function findInContextualBindings($abstract)
    {
        if (isset($this->contextual[end($this->buildStack)][$abstract])) {
            return $this->contextual[end($this->buildStack)][$abstract];
        }
    }
```

我们暂时还没看到 `contextual` 数组用过的地方，大家先看看就好。 并且输出 `contextual` 现在还是空数组，所以回到 `getConcrete `方法，判断不成立，继续往下走。

接着，判断我们上一章学过的 `bindings` 数组，我们知道数组已经包含了 `events` 这个单元了，

什么时候包含的？

是在 `application` 类的 `register` 方法内执行的 `$provider->register()` ，而 `events` 服务商类的 `register` 方法执行了 `$this->app->singleton`，而 `singleton` 其实在执行 `bind`，只不过是**共享绑定**而已，我们再复习一遍吧。

那也就是说，这里判断不成立，`events` 已经绑定好了，直接跳过 `if` 块，最后直接

```
    if (isset($this->bindings[$abstract])) {
        return $this->bindings[$abstract]['concrete'];
    }
```

我们知道，这里的 `concrete` 在 `events` 单元内是一个闭包对象，在把上一章的截图重发一下

![](https://dn-phphub.qbox.me/uploads/images/201510/10/1795/NdBLrcYpC9.png)

到此为止，`getConcrete` 理论上已经结束了，但是为了在弄清楚一点，我们继续把刚才 `if` 内没有执行的代码说一下：

咱们找到第一次能够执行此场景的代码，在入口文件 `index.php` ：

```
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
```

首先通过 `getAlias` 方法，知道现在 `$abstract` 参数是 `app\Http\Kernel`，`app\Http\Kernel` 到目前为止是没有经过 `bind`，所以判断成立，走 `if` 里面。

- `make`

```
    public function make($abstract, array $parameters = [])
    {
        return $this->resolve($abstract, $parameters);
    }
```

- `resolve`

```
    protected function resolve($abstract, $parameters = [])
    {
        $abstract = $this->getAlias($abstract);

        $needsContextualBuild = ! empty($parameters) || ! is_null(
            $this->getContextualConcrete($abstract)
        );

        // If an instance of the type is currently being managed as a singleton we'll
        // just return an existing instance instead of instantiating new instances
        // so the developer can keep using the same objects instance every time.
        if (isset($this->instances[$abstract]) && ! $needsContextualBuild) {
            return $this->instances[$abstract];
        }

        $this->with[] = $parameters;

        $concrete = $this->getConcrete($abstract);

        // We're ready to instantiate an instance of the concrete type registered for
        // the binding. This will instantiate the types, as well as resolve any of
        // its "nested" dependencies recursively until all have gotten resolved.
        if ($this->isBuildable($concrete, $abstract)) {
            $object = $this->build($concrete);
        } else {
            $object = $this->make($concrete);
        }

        // If we defined any extenders for this type, we'll need to spin through them
        // and apply them to the object being built. This allows for the extension
        // of services, such as changing configuration or decorating the object.
        foreach ($this->getExtenders($abstract) as $extender) {
            $object = $extender($object, $this);
        }

        // If the requested type is registered as a singleton we'll want to cache off
        // the instances in "memory" so we can return it later without creating an
        // entirely new instance of an object on each subsequent request for it.
        if ($this->isShared($abstract) && ! $needsContextualBuild) {
            $this->instances[$abstract] = $object;
        }

        $this->fireResolvingCallbacks($abstract, $object);

        // Before returning, we will also set the resolved flag to "true" and pop off
        // the parameter overrides for this build. After those two things are done
        // we will be ready to return back the fully constructed class instance.
        
        $this->resolved[$abstract] = true;

        array_pop($this->with);

        return $object;
    }
```

有3个知识点，`isBuildable` , `build` ,还有一个 `make` 的递归。

- - `isBuildable`，确定给定的具体物是可信赖的。

```
    protected function isBuildable($concrete, $abstract)
    {
        return $concrete === $abstract || $concrete instanceof Closure;
    }
```

要么具体物和抽象类型相等，要么具体物就是个闭包方法，至少要满足一样。

好，如果满足其中一样，我们现在场景的 `events` 是一个闭包，满足了，执行 `build`

- - `Build` 方法注释大概的意思：为给定的类型实例化一个具体物的实例。

方法开头直接:

```
    if ($concrete instanceof Closure) {
        return $concrete($this, $this->getLastParameterOverride());
    }    
```

终于露出你的庐山真面目了，`events` 闭包原型，我们在截图确认一下：

![](https://dn-phphub.qbox.me/uploads/images/201510/10/1795/EIAGw7jAmf.png)

嗯哼，`$this` 参数相当于闭包函数中的参数 `$app`，要注意的是 `$this` 是 `application` 类，不要认为是在 `container` 类定义的方法 `$this` 就是 `container`，它们是继承关系，而 `new` 的是 `application` 类，所以 `$this` 属于谁应该很清楚了。

然而就这样执行了闭包，里面的肯定是组件的一些功能，这也不是本章主题，所以略过。

至于这个 `make` 递归，暂时还不知道，等用到的时候在说，目前为止 `$object` 是一个实例了，我们截图看看是哪个类的实例即可，因为这些都是刚才闭包内执行的结果。

![](https://dn-phphub.qbox.me/uploads/images/201510/10/1795/pj8Sx0TxlN.png)

这个很简单，就是判断 `bindings` 数组单元内 `shared` 如果为共享状态（也就是1），则将 `$object` 进行赋值。

接下来有个 `fireResolvingCallbacks` 方法，我的理解是**启动“具有解析过程”回调函数**，目前没有执行到，我们也就不去说了，还是那句话，用的时候在具体来分析，更好理解。

`Make` 方法最后：

```
    $this->resolved[$abstract] = true;

    array_pop($this->with);

    return $object;
```

将 `true` 赋值到 `resolved` ,意为已经解析过的，最后 `return $object`,  make 结束。

## 总结

就目前为止，我们知道， `make` 会借用 `bindings`(bind而来) 来**获取具体的闭包对象**，然后用 `build` 方法来**调用闭包函数**，所以说 `bind` 和 `make` 是具有紧密的联系的


# 启动 `kernel`

前两章初步认识了 `bind` 和 `make` 使我们明白生成一个实例的步骤，接下来，我们要认识 `kernel`，这是与用户交互的关键，咱们打开入口文件

- 原文:

```
    /*
    |--------------------------------------------------------------------------
    | Run The Application
    |--------------------------------------------------------------------------
    |
    | Once we have the application, we can handle the incoming request
    | through the kernel, and send the associated response back to
    | the client's browser allowing them to enjoy the creative
    | and wonderful application we have prepared for them.
    |
    */
    
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );
```

- 翻译:

```
    /*
    |--------------------------------------------------------------------------
    | 运行 应用程序
    |--------------------------------------------------------------------------
    |
    | 一旦我们有了应用程序, 我们就可以 通过内核来处理传入的请求.
    | 并将相关的响应发送到客户端的浏览器
    | 让他们可以享受我们为他们准备的创意和精彩的应用程序
    |
    */
    
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );
```


咱们根据这行代码和这个参数，具体来捋一捋 `$kernel` 是如何生成的，也当复习。

`$app` 这个不用解释即 `application` 类，之前说的 `make` 方法是位于 `container` 类中，其实 `application` 类中也有 `make` 方法，如下：

```
    public function make($abstract, array $parameters = [])
    {
        $abstract = $this->getAlias($abstract);

        if (isset($this->deferredServices[$abstract]) && ! isset($this->instances[$abstract])) {
            $this->loadDeferredProvider($abstract);
        }

        return parent::make($abstract, $parameters);
    }
```

我们现在还不知道具体的用意在哪，不过我们看见了内部调用了父类( `container` )的 `make` 方法，

所以执行过程是先通过子类的 `make` 调用到父类的 `make` ，根据参数咱们继续往下看，这里只传了一个 `Illuminate\Contracts\Http\Kernel` 参数，所以 `$parameters` 可以无视了。

在继续分析父类 `make` 方法之前，我们要知道，在 `bootstrap/app.php` 里进行了3次 `singleton` 操作， `singleton` 前面也说过，也就是调用 bind 方法，如下：

```
    $app->singleton(
        Illuminate\Contracts\Http\Kernel::class,
        App\Http\Kernel::class
    );
    
    $app->singleton(
        Illuminate\Contracts\Console\Kernel::class,
        App\Console\Kernel::class
    );
    
    $app->singleton(
        Illuminate\Contracts\Debug\ExceptionHandler::class,
        App\Exceptions\Handler::class
    );
```

我们看见第一个 `singleton` 一目了然，对应了我们今天要说的类，不过后面多了一个参数（`app\Http\Kernel` ），这个参数原型名字是 `$concrete` ，即具体物。

了解以上这些，我们才正式分析父类的 `make` 方法：

`Make` 方法前一章单独说过了，所以略过没有执行到的代码。

这里我们知道，`Illuminate\Contracts\Http\Kernel` 还没有生成实例，只是**绑定**了而已，说到绑定，不知道大家还记得 `bind` 方法是怎么执行的吗？其中有一步是判断如果 `$concrete` 这个参数不是一个闭包的，则调用 `getClosure` 方法来强制生成一个闭包给 `$concrete` ，如下：

```
    protected function getClosure($abstract, $concrete)
    {
        return function ($container, $parameters = []) use ($abstract, $concrete) {
            if ($abstract == $concrete) {
                return $container->build($concrete);
            }

            return $container->make($concrete, $parameters);
        };
    }
```

咱们继续看 `container` 类的 `make` 方法。

```
    $concrete = $this->getConcrete($abstract);
```

我们知道这个方法只能返回2种参数，**一个是闭包对象，一个字符参数**，这里 `$concrete` 自然是得到一个闭包对象，因为已经绑定过了。

接着直接执行了 `build` 方法，咱们看 `build` 方法内的关键代码：

```
    if ($concrete instanceof Closure) {
        return $concrete($this, $this->getLastParameterOverride());
    }
```

- `getLastParameterOverride`

```
    protected function getLastParameterOverride()
    {
        return count($this->with) ? end($this->with) : [];
    }
```

嗯看到这里，我们上一章也说过，一个闭包的调用，这里需要跳回 `getClosure` 继续分析了：

`$concrete($this, $parameters)` 这里是传了一个 `this` 参数，即 `application` 类，所以 `$container=application` 类，那结合刚才说 `getClosure` 方法，这里闭包内部应该是执行一个 `$container->make($concrete)` 逻辑，注意参数 `$concrete` 是第一次传的 `app\Http\` 字符串；那更直接一点，代码其实是这样 `application->make(‘app\Http\Kernel’)` 。

从 `make(‘Illuminate\Contracts\Http\Kernel’)` 又转换到 `make(‘app\Http\Kernel’)` ，我们现在只需要知道这个顺序。

好，那再一次去看 `make` 如何实例化这个 `app\Http\Kernel` 类的，这次应该有不同的认识。

再次执行到 `getConcrete` 方法，我们说了要么是返回`一个闭包对象`，要么就是`一个字符串`，前面都是返回的闭包，而这次呢？转到 `getConcrete` 相关代码：

```
    protected function getConcrete($abstract)
    {
        if (! is_null($concrete = $this->getContextualConcrete($abstract))) {
            return $concrete;
        }

        // If we don't have a registered resolver or concrete for the type, we'll just
        // assume each type is a concrete name and will attempt to resolve it as is
        // since the container should be able to resolve concretes automatically.
        
        if (isset($this->bindings[$abstract])) {
            return $this->bindings[$abstract]['concrete'];
        }

        return $abstract;
    }
```


## Build 方法逻辑顺序：

先是我们最熟悉的闭包实例判断，我们这里只是一个字符串，所以不成立，再见。

```
    if ($concrete instanceof Closure) {
        return $concrete($this, $this->getLastParameterOverride());
    }
```

然后 `new` 了一个反射类

```
    $reflector = new ReflectionClass($concrete);
```

先是来了个 `isInstantiable` 判断，

```
    if (! $reflector->isInstantiable()) {
        return $this->notInstantiable($concrete);
    }
```

`PHP` 官网解释：

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/Ciz5dr12oy.png)

只要你不能实例化，直接抛错。

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/WmnBG5Ev0F.png)

接下来又学到一个新数组

```
    $this->buildStack[] = $concrete;
```

绑定栈，百度百科科普一下：

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/XL9V5n1Beo.png)

`app\Http\Kernel` 进入了栈结构了

紧接着又继续用到反射方法 `getConstructor` ，继续 `PHP` 官网：

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/8NQoEW5bJL.png)

```
    $constructor = $reflector->getConstructor();
```

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/ZqwZwsOT1R.png)

这下清楚了，如果反射不存在的话就返回一个 null 。

接下来也是一个 `is_null` 判断

```
    / If there are no constructors, that means there are no dependencies then
    // we can just resolve the instances of the objects right away, without
    // resolving any other types or dependencies out of these containers.
    
    if (is_null($constructor)) {
        array_pop($this->buildStack);
    
        return new $concrete;
    }
```

注释很明了，不需要多余的工作，先是出栈，弹出 `$concrete` ，然后直接返回对象实例。

这里 `app\Http\Kernel` 继承了父类 `Illuminate\Foundation\Http\Kernel` ，并且有构造函数，这里先发一下构造函数原型，以便后面更容易理解：

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/yxp8OFpGgg.png)

那么现在我们还得继续往下分析，发一下 `$constructor` 调试截图，更清晰一点：

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/bNksZTr6dg.png)

OK，那么现在 `$constructor` 是一个 `ReflectionMethod` 对象，这个反射方法对象我们在第二章也演示过，是针对类的方法。

紧接着:

```
    $dependencies = $constructor->getParameters();
```

- `getParameters` 方法示例：

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/S6Hb7woErL.png)

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/aOLf1dKNUM.png)

一环扣一环，返回值依然是一个新对象，不过是针对参数。

调试截图:

![](https://dn-phphub.qbox.me/uploads/images/201510/16/1795/ZOBmtPpgVQ.png)

嗯，确实匹配构造函数的参数，没什么大问题，每一个参数即是一个 `ReflectionParameter` 对象。

嗯哼，接下来也很关键：

```
    // Once we have all the constructor's parameters we can create each of the
    // dependency instances and then use the reflection instances to make a
    // new instance of this class, injecting the created dependencies in.
    
    // 一旦我们有了所以的构造函数的参数
    // 我们就可以创建一个依赖实例
    // 然后使用反射实例来创建这个类的一个新实例
    // 注入创建的依赖关系
    
    $instances = $this->resolveDependencies(
        $dependencies
    );
```

接着出栈:

```
    array_pop($this->buildStack);
```

最后很关键的地方来了:

```
    return $reflector->newInstanceArgs($instances);
    
    // 创建一个类的实例, 给出的参数将传递到类的构造函数
```

这个简单了吧，正好对应构造函数

```
    public function __construct(Application $app, Router $router)
```

这样解决的依赖关系.

## 总结

本章关键点在于 `build` 方法**利用反射解决依赖关系**那里，还有也进一步学习了 `getClosure` 方法内部逻辑，