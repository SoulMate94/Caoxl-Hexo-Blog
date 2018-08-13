---
title: 深入理解 Laravel
date: 2018-01-16 15:37:00
categories: Laravel
tags: [Laravel, PHP, 依赖注入, DI, IoC, Framework]
---

## 理解Laravel

> **建议深入理解的时候，不要看中文翻译版。**

在看 Laravel 手册的时候，关于系统架构的那几页篇幅始终看得懵懵懂懂的，今天得闲找了些资料看，把理解的成果记录一份在这里。


<!--more-->

## DI
> DI 依赖注入
 
### 作用

 - 避免在代码中硬编码类依赖，使在 runtime 和 compile time 期间改变依赖成为可能。
 - 由于实现了松耦合，项目变得：
   - 易维护
   因为不用再担心依赖的实现改变后，依赖的类也要适应性修改。
   
   - 方便单元测试
   当注入的是一个接口时，测试时可以很容易地创建一个实现该接口的测试类（mock）作为依赖传入，而不需要调用实际需要的类，避免一些单元测试期间并不希望做的实际操作。
   
   - 模块化
   类与类之间各司其职，不再关心依赖的实现细节。
   
### 实现方式

- 通过 constructor
```
    class C
    {
      // implementations...
    }
    class A
    {
    	private $c;
      
      	public function __construct(C $c)
        {
          $this->c = $c;
        }
    }
```

因为构造方法只会在实例化类的时候被调用，因此可以确保依赖一旦注入到实例后，在这个实例的 lifetime 期间，依赖始终存在，且不会被改变。

- 通过 setter
```
    class C
    {
      // implementations...
    }
    class B
    {
    	private $c;
      
      	public function setC(C $c)
        {
          $this->c = $c;
        }
    }
```

通过 setter 注入依赖是发生在类被实例化之后，这样的好处有：

- 方便注入那些可选的依赖，本类可以用默认值先创建出来，增加了灵活性。
- 简化了注入新依赖的过程：只要新增一个 setter 方法就行了，这并不会破坏现有代码，增强了便捷性和扩展性。

### 缺点

- 如果通过构造方法注入依赖
  - 则会强制性注入所有依赖，这不适合那些只是可选的依赖。（适合注入一些必须依赖）
  - 继承父类后，子类想要扩展和重写构造方法变得困难。

- 当项目规模增大时，虽然依赖注入实现了松耦合，但是这个解耦过程是通过手动来完成的，当依赖增多，手动注入依赖变成一项繁琐的工程。

### 什么时候需要依赖注入

大型项目。小项目使用过多设计模式反复杂化。


## IoC
> An IoC Container provides a central location to instantiate the application’s objects and pass their dependencies.
>
> If an object’s dependencies changes, this only needs to be set in one place, keeping your code DRY.

### IoC 容器， laravel 的核心

> OOP 的一种设计模式。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递给它。

Laravel 的核心就是一个 IoC 容器，根据文档，称其为“服务容器”，顾名思义，该容器提供了整个框架中需要的一系列服务。作为初学者，很多人会在这一个概念上犯难，因此，我打算从一些基础的内容开始讲解，通过理解面向对象开发中依赖的产生和解决方法，来逐渐揭开“依赖注入”的面纱，逐渐理解这一神奇的设计理念。

*本文一大半内容都是通过举例来让读者去理解什么是 IoC（控制反转） 和 DI（依赖注入），通过理解这些概念，来更加深入。更多关于 laravel 服务容器的用法建议阅读文档即可*


### 和 DI 的关系
> Dependency Injection is one and most used way to achieve IoC Pattern, another is Dependency Lookup.
>
> *An IoC Container is a convenience mechanism for achieving Dependency Injection. (Taylor Otwell)*

**简单来说: DI(依赖注入是配角),IoC(控制反转是主角)**

### IoC Container 核心原理

```
    class Container
    {
        protected $binds;
     
        protected $instances;
     
        public function bind($abstract, $concrete)
        {
            if ($concrete instanceof Closure) {
                $this->binds[$abstract] = $concrete;
            } else {
                $this->instances[$abstract] = $concrete;
            }
        }
     
        public function make($abstract, $parameters = [])
        {
            if (isset($this->instances[$abstract])) {
                return $this->instances[$abstract];
            }
     
            array_unshift($parameters, $this);
     
            return call_user_func_array($this->binds[$abstract], $parameters);
        }
    }
```

这是一个只有基础骨架的 IoC Container 实现，Laravel 的实现比这个更智能一点，但是基本原理是一样的。

> Laravel 的特性和功能几乎全部是由 IOC Container 实现的，如果想要使用 Laravel 的 IOC Container，也就是说想要用 IOC 的机制去 make 某种对象，那么你就必须先 bind 这个对象的类到 Laravel 的 IOC Container 中，才能把这种对象 make 出来。

需要注意的有：

- IoC Container 可以不止一个，所有继承了该类的子类，都属于 IoC Container，比如 Laravel 的 Application 类。
- Laravel 中的其他类都是先绑定到 IoC Container 之后，再通过 IoC Container 实例化，实现自动注入管理的。

## Service Container

> 服务容器
> > ServiceContainer 实现Contracts,具体的逻辑实现

Laravel 的核心，Service Container，其实就是一个 IoC Container，而无论叫什么名字，归根到实现底层实现，它们的目的只有一个：找到指定的类并实例化。

> 其实，我个人认为，现代 PHP 的很多核心概念，namaspace，PSR，autoload，Composer 等等，都是为了使「找到指定的类并实例化」这个操作变得简单化，自动化。

那么既然叫做 IoC Container 了，不同之处只是实现「找到指定的类并实例化」这个操作，是通过 IoC 的机制去实现的。

### 何为 Laravel IoC Container？

> A container is an object that, as you may expect, contains things.
>  
>  Laravel’s IoC container is used to contain many, many different bindings.
>  
>  Everything you do in Laravel will at some point have an interaction with the IoC container. This interaction is generally in the form of a binding being resolved.

所以我的简单理解就是，「服务容器」就是存放类绑定关系的容器。


### Laravel 使用 IoC Container 的两种方式
 - 不通过 Service Provider，直接使用现成的 IoC Container
 
 这个所谓的 「现成的 IoC Container」就是 Application 类，在 Laravel 启动后，bootstrap/app.php 中实例化了这个类，保存到变量 $app 了。
 
 先将类或接口手动绑定到 $app，通过 $app 中直接解析依赖并实例化对象。举例如下：
 
 ```
    <?php
    namespace App;
    // 1. define a class
    class A
    {
    	public $foo = 'bar';
    }
    // 2. bind class A to IoC container
    App::bind('a', function ($app) {
      return new A;
    });
    // 3. make instance from A with IoC container
    $a = App::make('a');
    echo $a->foo;    // bar
 ```
 
 其中，App 也是一个 Facade，是 Laravel 为了让我们在程序的各处都能方便的得到 $app，或者说 Application 类实例，或者说 Laravel 生命周期内的 IoC Container。
 
 - 通过 Service Provider（下面马上讲）
 

## Service Provider

> 服务提供者
> > ServiceProvider ServiceContainer的服务提供者，返回ServiceContainer的实例化，供其他地方使用，可以把它加入到app/config的provider中，会被自动注册到容器中

对，一个类要被容器所能够提取，必须要先注册至这个容器。既然 laravel 称这个容器叫做服务容器，那么我们需要某个服务，就得先注册、绑定这个服务到容器，那么提供服务并绑定服务至容器的东西，就是 `服务提供者（ServiceProvider）`

服务提供者主要分为两个部分，`register（注册）` 和 `boot（引导、初始化）`，具体参考文档。`register` 负责进行向容器注册“脚本”，但要注意注册部分不要有对未知事物的依赖，如果有，就要移步至 `boot` 部分。

### 什么是「服务提供者」？

> 我们知道，有时候我们的类、模块会有需要其他类和组件的情况，为了保证初始化阶段不会出现所需要的模块和组件没有注册的情况，Laravel 将注册和初始化行为进行拆分，注册的时候就只能注册，初始化的时候就是初始化。拆分后的产物就是现在的『服务提供者』。

我自己的简单理解就是，自动管理类依赖绑定、注册的机制。

这下其实就好理解了，「服务容器」，无论事先还是 runtime，存放了大量的类别名到实际类的绑定关系，而「服务提供者」的作用，可以将绑定关系存储到「服务容器」，并在某个类需要某个依赖的时候返回该依赖的实例。


### 为什么要通过「服务提供者」来使用 IoC Container？

上面讲到，完全可以不通过服务提供者来使用 IoC Container，为啥非要经过这么一个机制“饶一圈”呢？

其实还是一个量变引起质变的道理，当项目中只有少数几个类的时候，手动处理这些依赖关系是完全没问题的，但是当项目变复杂，变庞大的时候，人工解决这些依赖关系已经变得既累又不靠谱了，而把处理类与类之间的依赖关系这个过程，从手动变成自动，正是服务提供者的用处。


### 如何通过「服务提供者」来使用 IoC Container？

无论是否通过服务提供者来使用 IoC Container，都需要往服务容器中绑定好 key 和实际类的关系。只不过通过「服务提供者」是通过其的 register 系列方法完成了 bind，甚至是 make instance 的操作。

这些服务提供者，在 config/app.php 的 providers 数组中定义了一组服务提供者，在 app/Providers/ 目录下也定义了一些服务提供者，它们都有一个 `register()` 方法，通过服务提供者来使用服务容器就是通过这个 `register()` 方法来完成的。

比如，config/app.php => providers => Illuminate\Routing\RoutingServiceProvider：

```
    // ...
    // Register the service provider
    public function register()
    {
      $this->registerRouter();
      $this->registerUrlGenerator();
      $this->registerRedirector();
      $this->registerPsrRequest();
      $this->registerPsrResponse();
      $this->registerResponseFactory();
    }
    // Register the router instance
    protected function registerRouter()
    {
      $this->app['router'] = $this->app->share(function ($app) {
        return new Router($app['events'], $app);
      });
    }
    // ...
```

在服务提供者中，通过 $this->app 获得服务容器，然后注册一个 key 和实际类绑定。这里对于 RoutingServiceProvider 来说，顺便也返回了一个实例，即同时完成了 bind 和 make 操作。


## Contracts

> 直接翻译: 合同/契约
> > Contracts 合同，契约，也就是接口，定义一些规则，每个实现此接口的都要实现里面的方法

Contracts 就是 OOP 里的 Interfaces，功能和用途上没有任何区别，之所以叫 Contracts 只是因为在 Laravel 中，接口所在的命名空间是 Illuminate\Contracts 而已。

## Facades

> 直接翻译: 门面/外墙
> > Facades 简化ServiceProvider的调用方式，而且可以静态调用ServiceContainer中的方法

Facades 只是一些方便调用绑定在 IoC 容器内的类，的类。其作用我总结的是：简化调用方式，减少代码量。

具体解释在参考中解释的很清楚了，没必要重复。这里只以一条路由为例，简单总结下 Facade 的执行过程：


> 我们使用的 Route 类实际上是 Illuminate\Support\Facades\Route 通过 class_alias() 函数创造的 别名 而已，这个类被定义在文件 vendor/laravel/framework/src/Illuminate/Support/Facades/Route.php 。


- Route::get('/', 'Site@index')
- app/config/app.php => alias => Route => Illuminate\Support\Facades\Route::class
```
    amespace Illuminate\Support\Facades;
    /**
     * @see \Illuminate\Routing\Router
     */
    class Route extends Facade
    {
        protected static function getFacadeAccessor()
        {
            return 'router';
        }
    }
```

Route 类的这个方法会在后面「后期静态绑定」时用到。

- 调用了一个 Route／Facade 类中都不存在的静态 get() 方法 => 触发 Facade 的 __callStatic() 方法
```
    // namespace Illuminate\Support\Facades;
    // abstract class Facade
    // 1. trigger magic method `__callStatic` by call `get()`
    public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();
        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }
        switch (count($args)) {
            case 0:
                return $instance->$method();
            case 1:
                return $instance->$method($args[0]);
            case 2:
            	// `Route::get('/', 'Site@index')` 会执行到这里
                return $instance->$method($args[0], $args[1]);
            case 3:
                return $instance->$method($args[0], $args[1], $args[2]);
            case 4:
                return $instance->$method($args[0], $args[1], $args[2], $args[3]);
            default:
                return call_user_func_array([$instance, $method], $args);
        }
    }
    // 2. get facade instance
    public static function getFacadeRoot()  
    {
      	// 这里的 `static::getFacadeAccessor()` 属于后期静态绑定
        return static::resolveFacadeInstance(static::getFacadeAccessor());
    }
    // 3. get resolved instance from $app/IoC container
    protected static function resolveFacadeInstance($name)
    {   
        if (is_object($name)) {
            return $name;
        }
        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }
      
        return static::$resolvedInstance[$name] = static::$app[$name];
    }
```

这里，在第一次执行该代码的情况下，`resolveFacadeInstance()` 方法将 `static::$app[$name]` 保存到解析过的实例数组中，并返回 `static::$app[$name]` 的值。

这里之所以 `static::$app[$name]` 可以返回一个对象，是因为 `$app`/Application 继承了 Container，而 Container 又继承并实现了 ArrayAccess 的 offsetGet 方法：

```
    // namespace Illuminate\Container;
    // class Container
    public function offsetGet($key)
    {
      return $this->make($key);
    }
```

`__callStatic` 将执行到 `return $instance->$method($args[0], $args[1])`; 这里，而 `$instance` 就是 `offsetGet()` 返回的路由类对象，而 `$method` 就是那个 `Route／Facade` 都不存在的 `get()` 方法。于是，最终相当于执行了如下代码：

```
    (new Illuminate\Routing\Router)->get('/', 'Site@index')
```

- 终于，执行到 Illuminate\Routing\Router 的 get() 方法：

```
    public function get($uri, $action = null)
    {
      return $this->addRoute(['GET', 'HEAD'], $uri, $action);
    }
```

> suggestion: `class_alias()`.

**总的来说，自定义了一个类，为了方便在其他别处使用，便可以使用服务提供者和门面**

## 参考
 - [laravel 学习笔记 —— 神奇的服务容器](https://www.insp.top/learn-laravel-container)
 - [Digging in to Laravel’s IoC Container](https://code.tutsplus.com/tutorials/digging-in-to-laravels-ioc-container--cms-22167)
 - [从 1 行代码开始，带你系统性的理解 Laravel Service Container 的核心概念](http://blog.qiji.tech/archives/15626)
 - [Laravel应用](https://segmentfault.com/a/1190000004965752)