---
title: PHP  研究多一点
date: 2018-01-19 18:29:22
categories: PHP
tags: PHP
---

对编程来说，技术是招式，基础是内功，对自己有要求的程序员，就不能对自己使用的技术只是知其然而不知其所以然。

> - 知道怎么做，是一位合格开发者；知道为什么做，是一位优秀的程序员。
> - 基础不稳，面对技术复杂的系统，如同盲人摸象、管中窥豹，只得其门不得其法。（@rango）

下面总结的是使用 PHP 开发过程中，在实现相应功能之后，多的那么一些思考和研究。

> 鉴于本文的话题较广，因此本文会肯定会随着我研究的继续而定期更新。

<!--more-->


# 内核/底层

## SAPI

> In other words, SAPI is an **application programming interface** (API) provided by the web server to help other developers in extending the web server capabilities.

### PHP 常见的两种 SAPI

- Web/CGI SAPI

浏览器 => Web 服务器（Apache/Nginx）=> Web/CGI SAPI => PHP => Zend。

- CLI SAPI

CLI SAPI => PHP => Zend。

### CLI PHP 和 CGI PHP

CLI PHP 曾（3.0+）基于 CGI PHP ，但是直到 PHP 4.2.0 才从 CGI PHP 中独立出来。

两者功能非常类似，主要的区别在于：

- CLI 是 PHP 功能的基础部分
- CLI 没有只是 CGI 需要的和 Web 服务器相关的接口，不会导入 `$_GET`/`$_POST` 等变量，也不会输出 MIME 头信息
- CLI PHP 的运行机制和 Linux Shell 基本一样
- CLI PHP 有着和 CGI PHP 不同的默认值和 php.ini 设置

## 内存泄漏

`内存泄漏（Memory Leak）`，我个人简单的理解是：

*内存中的一块空间 A，在程序的所有作用域中已经没有任何变量指向 A（理应被标记为垃圾），但是却存在作用域外的另一个内存空间 B 对它保持着引用状态，B 会阻止垃圾回收机制（GC）回收 A，这种现象是内存泄漏。*

```
    $a   = ['v'];    // 产生一个变量容器
    $a[] = &$a;
    xdebug_debug_zval('a');
    
    --> Output:
    a:
    (refcount=2, is_ref=1)
    array (size=2)
      0 => (refcount=2, is_ref=0)string 'v' (length=1)
      1 => (refcount=2, is_ref=1)
        &array<
```

如果此时 unset 掉 $a，则会发生内存泄漏。

> 尽管不再有某个作用域中的任何符号指向这个结构(就是变量容器)，由于数组元素“1”仍然指向数组本身，所以这个容器不能被清除 。因为没有另外的符号指向它，用户没有办法清除这个结构，结果就会导致内存泄漏。
>  
> 庆幸的是，php将在脚本执行结束时清除这个数据结构，但是在php清除之前，将耗费不少内存。
> 
> 如果你要实现分析算法，或者要做其他像一个子元素指向它的父元素这样的事情，这种情况就会经常发生。
> 
> 当然，同样的情况也会发生在对象上，实际上对象更有可能出现这种情况，因为对象总是隐式的被引用。

## 扩展

### Swoole/PHP-X/EasySwoole

- https://wiki.swoole.com/
- https://www.easyswoole.com/

### 扩展开发

如果你有如下的应用需求，那么你可能会开始接触 PHP 扩展开发。

- 封装当前 PHP 尚不支持而有需要让 PHP 用到的 C/C++ 库
- 通过扩展重写一些性能较差的 PHP 代码
- 改善现有扩展
- 与其他语言编写的库交互

鉴于 PHP 扩展开发涉及到的东西较多，我会在其他的文章中再详细讨论，这里不详细讨论。

> https://stackoverflow.com/questions/645814/reading-a-git-repository-without-git

# 语法

## 可变函数参数

> 在 PHP 5.6 及以上的版本中，由 … 语法实现；在 PHP 5.5 及更早版本中，使用函数 `func_num_args()`，`func_get_arg()`，和 `func_get_args()` 。

- **使用 `…` 运算符定义变长参数函数**

```
    function f($req, $opt = null, ...$params) {
        // $params 是一个包含了剩余参数的数组
        printf(
          '$req: %d; $opt: %d; number of params: %d'."\n",
          $req,
          $opt,
          count($params)
        );
    }
    
    f(1);
    f(1, 2);
    f(1, 2, 3);
    f(1, 2, 3, 4);
    f(1, 2, 3, 4, 5);
```

- **使用 `…` 运算符进行参数展开**

```
    function add($a, $b, $c) {
        return $a + $b + $c;
    }
    
    $operators = [2, 3];
    echo add(1, ...$operators);
```

> 在其他编程语言，比如 Ruby中，这被称为连接运算符。

## 前期静态绑定与后期静态绑定

```
    class A
    {
        public static function foo()
        {
            self::who();    // PHP binds this to A::who() right away
            static::who();  // PHP waits to resolve `$this` (hence late)
        }
        public static function who()
        {
            echo __CLASS__, PHP_EOL;
        }
    }
    
    class B extends A
    {
        public static function test()
        {
            self::foo();
        }
        public static function who()
        {
            echo __CLASS__, PHP_EOL;
        }
    }
    B::test();
```

## nowdoc 和 heredoc

> nowdoc: PHP 5.3.0

两者用途都是为了给变量插入大段的字符串，语法基本相似，区别主要有两点：

- nowdoc 只是会单纯地输出大段字符串，而不会转换和解析字符串段中的任何变量。
- nowdoc 的字符串段区分符需要用单引号包裹起来。

**举例说明：**

```
    $var = 1024;
    $strNowdoc = <<< 'STR'
    ... large string blocks {$var} ...
STR;
    echo $strNowdoc;
    --> Output:
    ... large string blocks {$var} ...
      
    $strHeredoc = <<< STR
    ... large string blocks {$var} ...
STR;
    echo $strHeredoc;
    --> Output:
    ... large string blocks 1024 ...
```

## `??` 和 `?:`

`??` 是从 **PHP 7.0** 开始引入的语法糖，而 `?:` 是从 **PHP 5.3** 开始就有的。两者的区别是：

- `??`：如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。
- `?:`：表达式 expr1 ?: expr3 在 expr1 求值为 TRUE 时返回 expr1，否则返回 expr3。

## PDOStatement `bindValue` VS `bindParam`

```
    // bindParam()
    $sex = 'male';
    $s = $dbh->prepare('SELECT name FROM students WHERE sex = :sex');
    $s->bindParam(':sex', $sex); // use bindParam to bind the variable
    $sex = 'female';
    $s->execute(); // executed with WHERE sex = 'female'
    
    // bindValue()
    $sex = 'male';
    $s = $dbh->prepare('SELECT name FROM students WHERE sex = :sex');
    $s->bindValue(':sex', $sex); // use bindValue to bind the variable's value
    $sex = 'female';
    $s->execute(); // executed with WHERE sex = 'male'

```

## 构造函数

### PHP 无默认构造函数

PHP 没有默认构造函数。因此，如果调用类似 `parent::__construct()` 出现致命错误，可以先检查父类有无构造函数。

> See: <u>https://stackoverflow.com/questions/4650542/why-am-i-getting-fatal-error-when-calling-a-parents-constructor.</u>

### exit/die

如果在构造函数中使用了 `exit`或`die`，析构函数仍然会执行。

### 返回值

除非构造对象后显式地再调用一次构造函数可以拿到其返回值外（不推荐，属性可能丢失），一般情况下，直接使用 `new` 创建一个对象时，构造函数则会表现出“没有返回值”的现象

### 抛异常来产生错误

鉴于构造函数不能有返回值，因此如果在创建对象的时候如果因为一些初始化属性不合法想要产生错误，则最好使用抛异常的方式。

## 析构函数

### 无参数

析构函数不能有参数。

### 执行时机

- **对象被注销：** 不再有任何变量引用该对象。
- **请求结束：** PHP 执行引擎会在一个请求结束后调用析构函数。

### 用途

- 当对象被注销时记录一些日志信息。
- 链式调用时，为了保证链式调用的连贯性，可以在析构函数中执行一些默认行为。

**举例说明：**

```
    class View
    {
      	protected $html = '';
      	protected $outputed = false;
      
      	// render html only
        public function render(string $path) : View
        {      	
          	$this->html = include $path;
          
          	return $this;
        }
      	
        // output html only
      	public function output() : View
        { 
          	$this->outputed = true;
          
          	echo $this->html;
          
          	return $this;
        }
      
      	// 如果对象销毁时候视图模版还没有输出
      	// 则在析构函数中默认执行已渲染模版的输出
      	public function __destruct()
        {
            if (! $this->outputed) {
                $this->output();
            }
        }
    }
    $view = new View;
    $view->render('/path/to/file.php');    // will render and output template
```

### 注意事项

- PHP 不能准确给出对象析构函数被调用时的准确时间。对象被注销后，析构函数可能回延迟一段时间才被执行。
- 不要在析构函数中引用其他对象，因为其他对象的可用性不能保证，可能之前就被注销了。


## 浮点数精度问题

```
    // 问题示例：
    echo intval(17.9 * 100);   // 期望 1790 而实际输出 1789
    
    // 解决办法：
    echo intval(round(17.9 * 100, 0));    // output: 1790
    
    // Or:
    function getIntFee(float $float) : int
    {
      $arr = explode('.', $float);
      
      if (! isset($arr[0])) {
          return intval($arr[0]);
      }
      
      return false;
    }
    
    $amount = getIntFee(17.9 * 100); // output: 1790

```

## 生成器

### 判断命名空间是否存在

可使用 `class_exists()` 来检查命名空间是否存在.但是，但如果某些情况下报错 ：`undefined costant`，**可以检测命名空间开头是否有反斜线 \，若有则去掉**。

## Trait

> http://php.net/manual/zh/language.oop5.traits.php

- 概念&特点
  - Trait 是为了单继承语言而准备的一种**代码复用机制**。
  - Trait 和 Class 相似，它为传统的继承增加了水平的特性的组合，多个无关的 Class 之间不需要互相继承
  - Trait 使得无关的 Class 可以使用相同的属性和方法。
  - Trait 本身就是一个类的子集，不具备事先了解类成员是否冲突的能力。
  
PHP 5.4 以上便可以使用 Trait 特性。

- 优先级：当前类的成员高于 Trait 中的成员，而 Trait 则高于被继承的成员。
- Trait 间冲突

如果某个类引入的多个 trait 都包含了同名的方法，则会产生致命错误：

```
    <?php
    trait B
    {
        protected $name = 'cxl1';
      	public static $status = 1;
        function setName($name)
        {
            $this->name = $name;
        }
      
      	public static function staticMethod()
        {
          	echo 'trait can define static method too.';
    	}
      	public function setStatus($val)
        {
         	self::$status = $val; 
        }
      	public function getStatus()
        {
          	echo self::$status;
    	}
    }
    trait C
    {
        protected $name = 'cxl2';
        function setName($name)
        {
            $this->name = $name;
        }
    }
    class A
    {
        use B,C;
      
    	protected $name = 'cxl3';    // PHP 7.0.0 后没问题，之前版本是 E_STRICT 提醒
        function getName()
        {
            echo $this->name;
        }
    }
    $a = new A();
    $a->getName();
    // PHP Fatal error:  B and C define the same property ($name) in the composition of A. However, the definition differs and is considered incompatible. ...
    $a->setName('cxl3');
    // PHP Fatal error:  Trait method setName has not been applied, because there are collisions with other trait methods on A in ...
    $a->staticMethod();    // trait can define static method too.
    A::staticMethod();     // trait can define static method too.
    $a->getStatus();    // 1
    $a->setStatus(2);
    $a->getStatus();    // 2
```

Trait 定义了一个属性后，类就不能定义同样名称的属性，否则会产生 `fatal error`。 有种情况例外：属性是兼容的（同样的访问可见度、初始默认值）。(在 PHP 7.0 之前，属性是兼容的，则会有 E_STRICT 的提醒)

**方法冲突解决办法示例：**
    
```
    <?php
    
    trait B
    {
        function setName()
        {
            $this->name = 'b';
        }
    }
    
    trait C
    {
        function setName()
        {
            $this->name = 'c';
        }
    }
    
    class A
    {
        use B,C {
          	// 将 trait B 中的 setName() 重命名为 setNameB()
            B::setName as setNameB;
          	// 用 trait C 中的 setName() 替换掉 trait B 中的 setName()
            C::setName insteadof B;
        }
        protected $name = 'a';
        function getName()
        {
            echo $this->name;
        }
    }
    
    $a = new A();
    $a->getName();   // a
    $a->setNameB();
    $a->getName();    // b
    $a->setName();
    $a->getName();    // c
```

- **修改 trait 内方法的访问修饰符**

```
    <?php
    
    trait A
    {
        public function echoTrait()
        {
            echo 'trait';
        }
    }
    
    class B
    {
        use A {
            echoTrait as private;
        }
    }
    
    class C
    {
        use A {
            echoTrait as protected printTrait;
        }
    }
    
    $b = new B;
    $c = new C;
    $b->echoTrait();
    // PHP Fatal error:  Uncaught Error: Call to private method B::echoTrait() from context ...
    $c->echoTrait();
    // trait
    $c->printTrait();
    // PHP Fatal error:  Uncaught Error: Call to protected method C::printTrait() from context ...

```

- **trait with trait**

```
    <?php
    
    trait A
    {
        public function echoA()
        {
            echo 'a';
        }
    }
    
    trait B
    {
        public function echoB()
        {
            echo 'b';
        }
    }
    
    trait C
    {
        use A,B;
        public function echoC()
        {
            echo 'c';
        }
    }
    
    class Test
    {
        use C;
    }
    
    $test = new Test;
    $test->echoA();   // a
    $test->echoB();   // b
    $test->echoC();   // c

```

- **trait 中使用抽象方法约束类行为**

```
    <?php
    
    trait A
    {
        abstract public function necessary();
    }
    
    class B
    {
        use A;
        // 如果 B 没有此方法则会报错:
        // PHP Fatal error:  Class B contains 1 abstract method and must therefore be declared abstract or implement the remaining methods (B::necessary) ...
        public function necessary()
        {
            // something necessary ...
        }
    }
```

#应用

## 防止重复提交表单

- PRG：<u>https://stackoverflow.com/questions/10827242/understanding-post-redirect-get</u>
- 时间戳判断 过滤同一用户高频率的同一请求
- 执行非幂等操作前先判断数据是否已存在、已更新
- 客户端：点击提交表单后禁用按钮。

## 模版

PHP 本身就是一门模版语言，因此可以这么使用：

```
    <p>
        <?= $name, ' is ', $career ?>
    </p>
```

其中标签 `<?= ?>` 之中的 PHP 代码可以没有分号结尾，里面的所有变量默认会被当作字符串输出`（<?php echo）`，因此，如果不是标量类型的变量使用这种标签则会错误提示。

相反，`<?php ?>` 标签就是标准的 `PHP` 开始结束标签了，里面的 `PHP` 代码要完全符合正常 PHP 语法。

此外，`PHP` 还具有一个 **short_tag** 配置项，当它被设置为 `On` 时，可以使用更短的起始标签 **<? ?>** 来包裹 PHP 代码。*不过不推荐，因为这种标签会和 XML 冲突*。

## 时间日期

- 计算两个日期之间的天数

```
    $start = new Datetime('2017-01-01');
    $end   = new Datetime('2017-01-11');
    
    $interval = $end->diff($start);
    
    if (7 == $interval->format('%a')) {
        // do sth
    }
```

> 计算时间日期等最好使用可靠的库，手动计算容易出错。


# 设计/思想

> 探索技术背后的原理，学习技术背后的设计思想，才是偷不走的不可替代性。

## DDD

Domain driven deign。
> 领域驱动设计。

## 微服务

- <u>PHP-MSF开发手册</u>


# 环境

## CGI/FastCGI／PHP-FPM
> PHP-FPM(FastCGI Process Manager：FastCGI进程管理器)是一个PHPFastCGI管理器

`PHP` 解释器实际上只有 `php-cgi`，即只是一个单纯的 `CGI` 程序，`php-cgi` 的工作原理很纯粹，能且只能干：

- 解析 php.ini 文件，初始化执行环境
- 解析请求
- 返回结果

其中，第一步中的“初始化执行环境”是 PHP 早期产生性能问题的主要原因，为了解决这个问题，fastcgi 协议改善了上述工作流程：

> “首先，Fastcgi会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理”。

修改 php.ini 之后，php-cgi 进程是没办法平滑重启的。php-fpm 能够实现“平滑重启”的原理也很简单：**新的 `php-cgi worker` 用新的配置，已经存在的 php-cgi worker 处理完本次请求后就退出**。

## nginx+php-fpm 自定义 HTTP Header

添加到 HTTP 请求中的自定义 `Header` 格式必须为 `A-B-C`（-／英文横线），而 PHP `$_SERVER` 变量得到的就会是：`HTTP_A_B_C`。

其他格式则不会被正确获取到，除非修改 `nginx` 配置 `underscores_in_headers` 为 `On` ，See：[Module ngx_http_core_module#underscores_in_headers](http://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers)。

## 生产环境常用配置

### zend opcache（不能同时使用 zend guard loader）

> Zend OPcache 通过 opcode 缓存和优化提供更快的 PHP 执行过程。它将预编译的脚本文件存储在共享内存中供以后使用，从而避免了从磁盘读取代码并进行编译的时间消耗。同时，它还应用了一些代码优化模式，使得代码执行更快。
>  
> 当解释器完成对脚本代码的分析后，便将它们生成可以直接运行的中间代码，也称为操作码（Operate Code，opcode）。Opcode cache 的目地是避免重复编译，减少 CPU 和内存开销。如果动态内容的性能瓶颈不在于 CPU 和内存，而在于 I/O 操作，比如数据库查询带来的磁盘 I/O 开销，那么 opcode cache 的性能提升是非常有限的。但是既然 opcode cache 能带来 CPU 和内存开销的降低，这总归是好事。
> 
> 现代操作码缓存器（Optimizer+，APC2.0+，其他）使用共享内存进行存储，并且可以直接从中执行文件，而不用在执行前“反序列化”代码。这将带来显着的性能加速，通常降低了整体服务器的内存消耗，而且很少有缺点。

```
    opcache.enable=1
    opcache.enable_cli=1               ; 是否在 CLI 模式下启用 opcache
    opcache.memory_consumption=128     ; 共享内存的大小
    opcache.max_accelerated_files=2000 ; 最大缓存文件个数
```

## php-fpm

> <u>http://php.net/manual/zh/install.fpm.configuration.php</u>

- 工作进程分配模式

在 fasgcgi 模式下，php-fpm 会启动多个子进程，来处理 nginx 发来的请求。

```
    pm = static | dynamic | ondemand
```

**static 模式：** 表示启动时创建的 php-fpm 子进程数量是固定的，此时只有 pm.max_children 这个参数生效。

**dynamic 模式：** 表示启动的子进程数是有请求量动态变化的，受 `pm.max_children／pm.start_servers`/`pm.min_spare_servers`/`pm.max_spare_servers` 共同决定。

工作模式选择原则：**小内存机选动态，省内存；大内存机选静态**。

不过，动态模式下，进程的动态创建和回收本身也需要占用服务器资源。

> 如果你的内存很大，有8-20G，按照一个php-fpm进程20M算，100个就2G内存了，那就可以开启static模式。如果你的内存很小，比如才256M，那就要小心设置了，因为你的机器里面的其他的进程也算需要占用内存的，所以设置成 `dynamic` 是最好的，比如：`pm.max_chindren = 8`, 占用内存`160M`左右，而且可以随时变化，对于一般访问量的网站足够了。

- 慢日志

```
    slowlog = var/log/php-fpm.log.slow    # 必须在 request_slowlog_timeout 前定义
    
    request_slowlog_timeout = 10s         # 当一个请求时间超过 10 秒后 将对应的 PHP 调用栈写到慢日志
```

# 性能

> TODO...

# FAQ

- **<u>[How to Match A Backslash with preg_match() in PHP?](https://www.developwebsites.net/match-backslash-preg_match-php/)</u>**

use `\\\\` instead of `\\`。（Regex Tester）

- **headers already set**

> <u>https://stackoverflow.com/questions/8028957/how-to-fix-headers-already-sent-error-in-php.</u>

- **curl 工作不正常？**

curl URI 中含有空格会请求失败。

# 参考

## Book

- <u>[PHP7内核剖析-pangudashu](https://github.com/pangudashu/php7-internal)</u>
- <u>[PHP Internals Book](http://www.phpinternalsbook.com/index.html)</u>

## Link

- [What is SAPI and when would you use it?](https://stackoverflow.com/questions/9948008/what-is-sapi-and-when-would-you-use-it)
- [Where can I learn about PHP internals? [closed]](https://stackoverflow.com/questions/4389738/where-can-i-learn-about-php-internals)
- [Zend API: Hacking the Core of PHP](http://php.net/manual/en/internals2.ze1.zendapi.php)
- [PHP7扩展开发教程[1] – 怎样导出一个模块？](https://yuerblog.cc/2017/08/07/course1-how-to-export-a-module/)
- [motecshine/php-ext-design-patterns](https://github.com/motecshine/php-ext-design-patterns)
- [PHP The Right Way](http://www.phptherightway.com/)
- [PHP Best Practices-A short, practical guide for common and confusing PHP tasks](https://phpbestpractices.org/)
- [when using self, parent, static and how?](https://stackoverflow.com/questions/10504129/when-using-self-parent-static-and-how)
- [php exec() is not executing the command](https://stackoverflow.com/questions/17914402/php-exec-is-not-executing-the-command)
- [Post/Redirect/Get-Wikipedia](https://en.wikipedia.org/wiki/Post/Redirect/Get)
- [腾讯PHP工程师面试题两份](http://www.cnblogs.com/52php/p/5666337.html)
- [PHP interfaces IteratorAggregate vs Iterator?](https://stackoverflow.com/questions/13624639/php-interfaces-iteratoraggregate-vs-iterator)
- [PHP’s nowdoc strings](https://www.electrictoolbox.com/php-nowdoc-string/)
- [What is the difference between bindParam and bindValue?](https://stackoverflow.com/questions/1179874/what-is-the-difference-between-bindparam-and-bindvalue)
- [Implementing Domain-Driven Design in Laravel](https://www.developer.com/open/implementing-domain-driven-design-in-laravel.html)
- [php高并发知识栈](https://www.habby.top/archives/52.html)
- [php-fpm参数优化](https://blog.linuxeye.cn/380.html)
- [php-fpm.conf & php.ini 安全优化实践](https://klionsec.github.io/2017/11/23/phpsec/)
- [php-fpm的配置和优化](https://www.zybuluo.com/phper/note/89081)
- [分析 PHP 应用程序以查找、诊断和加速运行缓慢的代码](https://www.ibm.com/developerworks/cn/opensource/os-php-fastapps2/index.html)
- [PHP高效率写法（详解原因）](http://www.open-open.com/lib/view/1332904714233#_label0)

