---
title: PHP 项目优化
date: 2018-01-24 16:44:34
categories: PHP
tags: [PHP, 优化]
---

关于 PHP 性能分析、优化和实践总结在这里。

> 无论现在有没有遇到性能问题，但是这是迟早都会面临的问题。

<!--more-->


# 性能优化概述

## 影响 PHP 性能的可能性

- 语法使用不当
- 使用 PHP 做了它不擅长的事情

PHP 适合处理服务端业务逻辑、数据库处理、和视图渲染，不适合计算密集型业务，不适合表示复杂的算法。

- 通过 PHP 连接的服务有性能问题

数据库，第三方服务接口等等。

- PHP 语言本身的缺点或特点

  - 语法不严谨
  
  严格来说也不叫缺点，而应该是特点
  
  这个“缺点”会让经验不足、水平不够的程序员写出糟糕的代码。
  
  - 原生对多线程支持不好
  
  通常只能模拟实现简单的多线程场景。
  
  - 原生不支持对象常驻内存
  PHP 所有变量都是页面级的，PHP 页面执行完毕就会被释放。
  
- PHP 周边环境问题

比如操作系统、内存、磁盘I/O、网络环境、Web 服务器、DB 等本身的一些限制。

> 整个项目的性能问题不可能都是 PHP 的性能问题，因此优化 PHP 项目性能时，不要局限于优化 PHP。

- 网络问题

- 一些未知因素

## 优化 PHP 项目的方向

- PHP 代码级别
- PHP 周边问题
- PHP 语言自身

从上往下，难度依次增加。


# PHP 代码层优化

## PHP 内核

标准的 PHP 内核简单说只有 3 个：

- 词法分析
- 语法分析
- Zend Engine

从宏观上来看，PHP 内核的实现与世界上绝大多数的程序一样：接收输入数据， 做相应处理，然后输出计算结果。我们编写的代码就是 PHP 接收的输入数据，PHP 内核对我们编写的代码进行解释和运算， 最后返回相应的运算结果。

> 然而，PHP 与我们自己平时写的一般的 C 程序有所不同的是， 我们的程序一般用来解决某个具体问题， 而 PHP 本身实现了把用户的逻辑“翻译”为机器语言来执行的功能，这也是各种编译语言与承载具体业务逻辑的程序代码的一个明显区别。

## Web 应用中 PHP 代码运行流程

> 之所以要加上 “Web 应用中” 这个前提，是因为 PHP 除了常用的处理 Web 请求外，还可以在 CLI 模式下运行，而两种运行方式的具体实现是不一样的。

从 HTTP 请求到达一个 PHP 脚本开始，到 PHP 处理结束返回结果之间，标准 PHP (<u>https://php.net</u>) 会经历以下过程：

- *.php 文件

PHP 内核接受 PHP 代码。

- Scanner：词法分析

词法分析是把 PHP 代码分割成一个个的“单元”（TOKEN）：Zend 引擎逐行运行扫描、分析，结束后保存为 Zend 能识别的代码 Exprs。

- Parser：语法分析

语法分析则将这些“单元”转化为 Zend Engine 可执行的操作：Exprs 被 Parser 解析为 Opcodes，即最终会被 PHP 内核第三个组成部分 Zend Engine VM，执行的代码

- Exec

Zend Engine VM 执行 Opcodes。

> PHP 现有的很多缓存服务都是直接缓存的 Opcode，这也是为了提高 PHP 的性能。

- Output

输出执行结果。

## 多用 PHP 自身能力

**能用 PHP 内置方法解决的就用内置方法，不要重复造轮子。**

自写代码一般冗余较多，可读性不佳，主要是性能低。因为自写代码需要编译解析为底层语言才能被执行，这个过程每次请求 PHP 页面时都会发生，时间开销大。

因此，多使用 PHP 内置变量、常量、API、解决方案。

**举例说明：**

```
    vim -o bad.php good.php
```

- bad.php

```
    <?php
    
    $arr1 = array();
    $arr2 = array();
    
    for ( $i=0; $i<rand(1000, 2000); ++$i ) {
      $arr1[] = rand();
    }
    
    for ( $i=0; $i<rand(1000, 2000); ++$i ) {
      $arr2[] = rand();
    }
    
    $arr_merged = array() ;
    foreach ( $arr1 as $val ) {
      $arr_merged[] = $v;
    }
    
    foreach ( $arr2 as $val ) {
      if(!in_array($val, $arr_merged)){
    	$arr_merged[] = $val ;
      }
    }
    
    var_dump( $arr1, $arr2, $arr_merged ) ;
```

- good.php

```
    <?php
    $arr1 = $arr2 = range( 1000, 2000 ) ;
     
    shuffle( $arr1 ) ;
    shuffle( $arr2 ) ;
    $arr_merged = array_merge( $arr1, $arr2 ) ;
    var_dump( $arr1, $arr2, $arr_merged ) ;
```

bad.php 和 good.php 都是为了完成合并数组，并要求重复的只出现一次这个操作。但是，两者的性能却差别巨大，可以使用压力测试：

```
    ab -n100 -c10 http://localhost/bad.php
    ab -n100 -c10 http://localhost/good.php
```

结果显示，good.php 比 bad.php 的性能高 > 10 倍。

bad.php 和 good.php 相比，虽然在执行的时候虽然都会经历上面几个流程，但是，在逐行扫描的过程中，直接用 PHP 内置函数会节省更多的时间，因为 **PHP 内置函数 Zend 引擎是理解的**，就不会耗费时间进行分析，从而极大地提高了速度。

总之，使用 PHP 自身方法去实现一个功能，比用 PHP 自写代码自定义方法实现同样功能，性能要高很多，所以要尽可能使用 PHP 函数。

## 使用性能更好的 PHP 内置函数

PHP 内置函数之间，仍然存在快和慢的差距。要从代码层优化 PHP 项目，多了解 PHP 内置函数的时间复杂度也是必不可少的。

以 `isset()` 和 `array_key_exists()` 方法对比说明：

```
    $start = getTime();
    $i = 0;
    $arr = range(1, 200000);
    
    while ($i < 200000) {
      ++$i;
      // 1. isset()
      // isset($arr[$i]);
      
      // 2. array_key_exists()
      array_key_exists($i, $arr);
    }
    
    $end = getTime();
    echo 'Spent: ', number_format($end-$start, 3)*1000, 'ms', PHP_EOL;
    
    // 获得微秒级别的 UNIX 时间戳
    function getTime() {
    	list($usecs, $secs) = explode(' ', microtime());
    	return (float)$usecs + (float)$secs;
    }
```

运行后发现：`isset()` 比 `array_key_exists()` 效率更高大约 10 ms。

## 尽可能少用魔术方法

这是因为 PHP 提供的魔术方法性能都不怎么好。

至于为什么不好，这个理性地解释应该是，为了省事，PHP 为了程序员做了很多。举例说明：

```
    // test.php
    // 1. 使用魔术方法
    class Test
    {
      private $var = 1024 ;
      public function __get( $var )
      {
      	return $this->var ;
      }
      public $var = 1024;
    }
    
    // 2. 不使用魔术方法
    class Test
    {
      public $var = 1024;
    }
    
    # 测试魔术方法
    $i=0;
    while ( $i < 10000 ) {
      $i ++;
      $test = new Test();
      $test->var;
    }
```

可以使用 time 命令来简单判断执行时间：

```
    time php test.php
```

测试结果显示，在相同测试条件下，不使用魔术方法比使用魔术方法效率更高（这个例子的规模下，相差近 3 倍）。

## 避免使用 @

PHP 提供错误抑制符，也是为了方便懒人。

> 在每次”方便”之间，损失的是性能。

@ 的实际工作原理是：

  - 先降低报错等级，再恢复报错等级。
  - 在代码开始前，结束后，增加 Opcode，忽略报错。

可以使用 vld 工具（下面会说）测试。举例如下：

- test.php

```
    <?php
    @file_get_contents( 'xxx' );    // wrong path
```

- vld 测试

```
    # active=1 表示使用 vld
    # execute=0 表示不执行代码只是查看 Opcode
    # verbosity 代表输出VLD 生成内容的详细程度
    
    php -dvld.active=1 -dvld.execute=0 test.php
```

建议：尽量使用 `try {} catch ()` 的方式控制错误而不要使用这种取巧的方式。

## 合理使用内存

虽然 PHP 提供了 GC 机制，但是不意味着放心大胆地使用内存。

**建议：** 使用 `unset()` 及时释放不使用的内存。

## 尽量少用正则

正则表达式的回溯开销大，且逻辑复杂，容易出错。

**建议：** 能用字符串函数等 PHP 内置 `API` 解决的问题就不要用正则表达式。

## 避免循环中动态计算

动态计算指的就是函数调用。典型的坏例子：

```
    $str = 'i am a bad example.';
    for ($i=0; $i<strlen($str); $i++) {
      // ...
    }
```

这种情况，就算 `strlen()` 效率再高，也是无意义的浪费。

但是，也不是说循环内就不能调用函数了，一些不是重复的数据，在特定的使用场景下，也不得不在循环内调用函数计算，这和这个例子不一样。

> PHP 不适合处理大数据，如果使用 PHP 处理大数据，那么很容易出现性能瓶。
> 
> PHP 配合模板引擎适合 UI 呈现。
> 
> PHP 的所有计算都将转换为 C 语言，很多代码实现都依赖 C 实现。

## 使用 PHP Generator

> PHP generators are an underutilized yet remarkably helpful feature introduced in PHP 5.5.0.
> 
> Generators are simple iterators.
> 
> Generators compute and yield iteration values on-demand.

```
    function myGenerator() {
        yield 'value1';
        yield 'value2';
        yield 'value3';
    }
    
    foreach (myGenerator() as $yieldedValue) {
        echo $yieldedValue, PHP_EOL;
    }
```

但是，PHP Generator 只能用于单方向迭代的、一次性的需求，但是这种需求的出现频率还是挺高的。关于用和不同的差距，可以举一个例子对比：

- bad.php

```
    function makeRange($length) {
        $dataset = [];
        for ($i = 0; $i < $length; $i++) {
            $dataset[] = $i;
        }
        return $dataset;
    }
    
    $customRange = makeRange(900000);
    foreach ($customRange as $i) {
        echo $i, PHP_EOL;
    }
```

- good.php

```
    function makeRange($length) {
      for ($i = 0; $i < $length; $i++) {
            yield $i;
        }
    }
    
    foreach (makeRange(900000) as $i) {
        echo $i, PHP_EOL;
    }
```

可以分析两者的执行结果，结论是：goods.php 比 bad.php 不仅提升性能，而且节约了内存，解决了迭代数据量过大时内存不够的问题。

在 bad.php 中，如果把规模调大，比如 100000000，那么执行 bad.php 时，其实是没有输出的，因为超过了 PHP 配置中分配给脚本所能使用的上限（`memory_limit`），但是使用 good.php 却能正常显示，这是因为 `yield()` 方法是按需生成数据的，占不了多少内存。

建议：能用 Generator 替换默认迭代的就使用 Generator。

## 数组键名为常量问题

```
    define( 'key', 'value' );
    
    $arr = array(
      'key'   => 'hello' ,
      'value' => 'world'
    );
    
    echo $arr[ key ], PHP_EOL;
    echo $arr[ 'key' ];
```

PHP 会把不带引号的数组键当作常量，从而先去常量表中查找该常量对应的值，只有找不到的时候才会在数组内部中找。

不过这种“习惯”既不有利于性能，也不有利于程序的健壮性。因此，尽量不要使用常量作为数组的键名。


# PHP 周边环境优化

## PHP 的周边环境

- 操作系统性能（光是这一个就有很多影响因素：版本、文件系统、系统软件等等）
- 硬盘性能
- 内存 / 缓存
- 数据库软件（数据库基于文件系统）
- Web 服务器
- 网络状况（带宽、传输介质）

## PHP 开销和速度排序

- 开销由大到小

磁盘 I/O > 数据库 I/O > 内存 I/O > 网络 I/O。

- 速度由快到慢

内存 I/O > 数据库 I/O > 磁盘 I/O > 网络 I/O。

由于数据库软件既可以从内存中读写也可以从磁盘中读写，所以其性能介于内存和磁盘之间。

读写网络数据本质也是磁盘操作（操作 socket 文件句柄），但是网络都是有延迟的，延迟带来了隐性时间浪费。

总结：尽量减少对磁盘和网络数据的操作，尤其是慢速网络接口的大数据，尽量使用内存和数据库。

## 数据库性能是 Web 项目性能优化的重点

PHP 脚本是串行执行的，而 PHP 脚本本身的执行时间是比较短的，因此大部分时间都在等待， 尤其往往是在等待网络另一端的数据库。

由于主要的耗时在数据库，因此优化的时候重心应该在对数据库自身操作的优化，而不是单单考 虑 PHP 语言自身的问题。（关于数据库性能，详见另一篇文章）

建议：把更多的计算任务交给 PHP 而不要在数据库中进行多余的计算。

## 网络请求优化

### 网络请求的特点

- 对方接口的不确定因素
- 网络是否可达
- 网络稳定性
- 数据格式是否是期望的

### 如何优化网络请求

- 设置超时时间
  - 连接超时：200ms
  - 读超时：800ms
  - 写超时：500ms

可根据情况稍微增大超时时间，但是再大不能大于 1 秒。

- 将串行请求并行化

  - 使用 `curl_multi_*()`
  
  但是 *curlmulti* 扩展依赖于最慢的请求响应时间，所以尽管并行化，但是最终的等待时间仍然是所有并发请求中最慢的那一个，一旦有个响应变慢就会拖慢整个响应。
  
  - 使用 swoole 扩展
  
  `swoole` 是一个 php 扩展，是从 `C` 语言层面实现并行化。
 
- 压缩 PHP 接口输出

  - 如何压缩：一般服务器启用 gzip 即可。
  
  - 压缩的利弊：利于数据输出，减少客户端获取数据速度；缺点是会产生额外的 CPU 开销（也会使客户端产生 CPU 开销，因为要解压）。
  
  综合考虑，当接口数据小于几十 KB 时，压缩带来的效果并不明显，甚至比以前更大但是当大于 100 KB 时， `Gzip` 能够压缩到 30 KB / 40 KB 左右 此外，如果数据内容中重复的数据越多，压缩效率越高，反之越不明显。

## 缓存重复计算内容

- 什么时候需要缓存内容？

多次请求同一内容时

- 缓存原理

请求某个页面时，首先从缓存中查找是否存在。如果命中，则直接从缓存中返回；如果缓存中没有，则重新计算，并把计算结果保存在缓存中，最后返回给客户端。

## 流程优化

可以将一个串行的流程分解为以下两种并行流程：

### 重叠时间窗口

只有在后一个任务不以前一个任务为强依赖的时候才采取这种思想。

### 旁路方案

旁路方案和重叠时间窗口是非常类似的，因此也有相同的限制条件：前后任务不具有依赖关系。

> 前面网络请求的复用，就是采用了重叠时间窗口的思想。
>
> Smarty 在模版渲染的时候就采用了缓存和旁路方案。


# 关于 PHP 语言自身瓶颈

## Opcode 优化

优化思路是缓存 Opcod，因为 Opcode 是 PHP 代码在执行过程中最接近与机器码的代码。

> 这个可用 APC 扩展解决。不过 APC 从 2012 年开始就已经不再更新了，不过可以在上面搜索 Caching 查找相关缓存类扩展方案，比如知名的 memcache/memchached、vainish、vac (@Xinchen Hui @Wei Dai[蘑菇街]）。
>  
> 共享内存自认为是替代 APC 和本地 Memcache 的方案 。
> 
> PHP 的扩展实现的目标：通过扩展代替原 PHP 代码中的高频逻辑。

## Runtime 优化

这个目前只能用 HHVM 和 PHP 7 解决。

> HHVM 是由 FaceBook 开源的技术，火了一段时间，百度也使用过该技术。

**总结：** PHP 的性能优化思路不仅仅局限于 PHP 层面的优化，还包括许多内容，需要综合考虑，必 要时借助工具帮助分析，找到瓶颈所在，找出解决之道。


# 性能分析工具

## AB

<u>[Apache Benchmark](https://httpd.apache.org/docs/2.4/programs/ab.html)</u> 是经常用到的压力测试工具。一般安装 Apache 服务器时会自动安装好该工具。如果没有，可以手动安装：`yum install httpd-tools`。

使用方式举例如下：
    
```
    # 使用 100 次并发模拟 1000 次请求
    ab -n1000 -c100 https://www.myapp.com/
    
    # 在 10 秒内模拟 1000 次并发请求
    ab -t 10 -c 1000 https://www.myapp.com/
```

需要注意的是：URL 末尾一定要带上 `/`，否则会提示非法 `URL`。

测试结果中需要关注的参数有：

- requests per second

每秒接受的请求数，优化的方向是越多越好，并发能力要高。

- time per request

响应每次请求的毫秒数，优化的方向是越短越好，响应时间要小。

> 本人测试 `https://caoxl/com/` 后 使得网站https失效~~

## XHProf

<u>[XHProf](https://pecl.php.net/package/xhprof)</u> 是 Facebook 开发的 PHP 性能分析工具，以 PHP PECL 扩展的形式安装使用。

### 下载&安装

```
    # 下载
    wget http://pecl.php.net/get/xhprof-0.9.4.tgz
    
    # 解压
    tar zxvf xhprof-0.9.4.tgz
    
    # 编译安装（确保 php5-dev 已经安装）
    cd xhprof-0.9.4/extension/
    ./configure
    # ./configure --with-php- config=/usr/local/php/bin/php-config
    make && make install
```

注意，在 make 过程中，如果遇到 `warning` 和 `error`，只需找到出错的文件和位置，可以尝试注释，然后重新执行：`make clean && make`。

安装成功后会看到提示：*Installing shared extensions: /usr/lib/php5/20131226/*。

如果是 64 位系统，则可能需要将 `xhprof.so` 拷贝到相关的 `lib64` 目录下。

### 配置 & 重启

```
    cd /etc/php5/fpm/conf.d
    vim 20-xhprof.ini
        extension=xhprof.so 
        xhprof.output_dir=/var/www/html/xhprof/
        
    # 重启
    /etc/init.d/php5-fpm restart
    
    # 查看是否安装 xhprof 扩展
    php --ri xhprof
```

### xhprof web gui

详细查看：<u>https://github.com/phacility/xhprof。</u>

### XHPorf 使用举例

- cpu：`xhprof_enable(XHPROF_FLAGS_CPU)`
- 内存：`xhprof_enable(XHPROF_FLAGS_MEMORY)`
- 如果两个一起：`xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY)`

```
    // 1. enable xhprof
    xhprof_enable();
    
    // 2. code to be tested here
    require '/path/to/script';
    
    // 3. disable xhprof
    $data = xhprof_disable();    // 返回运行数据
    
    // 4. incloude lib (xhprof_lib 在下载的包里存在这个目录, 需要将目录包含到运行的 php 代码中)
    include_once '/path/to/xhprof_lib/utils/xhprof_lib.php';
    include_once '/path/to/xhprof_lib/utils/xhprof_runs.php';
    
    $xhprofObj = new XHProfRuns_Defatult();
    $run_id = $xhprofObj->save_run($data, 'test');    // 第一个参数是 xhprof_disable() 函数返回的运行信息; 第二个参数是自定义的命名空间字符串(任意)
    
    // 返回运行ID, 用这个 ID 查看相关的运行结果
    var_dump($run_id);
```

请求该页面，然后打开 xhprof 的 VIEW 层展现工具就可以看到详尽的报表和流程图信息。

**报表信息说明：**

- Incl. Wall Time

代表的是流程走到这个函数的时候，当前函数的执行时间，和当前函数之后所有函数的执行时间的总微秒数。

- Excl. Wall Time

一个函数的执行时间（微秒）。

- 一个函数的执行时间（微秒）。

展示完整的项目请求流程图，`黄色、红色和粗的路线`代表的是耗时较多的操作和流程。

假设找到了最耗时的文件，比如为 `_import_fromheader`，就可以打开源码进行排查：

```
    cd /path/to/proj
    grep 'import_from_header'
```

然后阅读源码进行分析，debug，之后再次重复上述操作看优化是否生效，以及决定是否继续进行优化。

## vld

> <u>[Vulcan Logic Dumper](http://www.phppan.com/2011/05/vld-extension/)</u>，是一个在 Zend引擎中，以挂钩的方式实现的，用于输出 PHP 脚本生成的中间代码（Opcode，执行单元）的扩展。 它可以在一定程度上查看 Zend 引擎内部的一些实现原理，是我们学习PHP源码的必备良器。

### 下载&&安装

```
    # Centos
    yum search php5-devel
    yum install php5-devel
    
    # Or Debian
    # apt-get search php5-dev
    # apt-get install php5-dev
    
    git clone https://github.com/derickr/vld.git vld
    # Or
    # wget http://pecl.php.net/package/vld/0.13.0
    
    tar zxvf vld-0.13.0.tgz
    cd vld-0-13.0
    phpize
    ./configure
    make && make install
```

### 配置

- Nginx

可以通过 `phpinfo()` 中的 `Additional.inifilesparsed` 属性来找到 PHP5-FPM 读取配置文件的路径，然后创建配置文件:
    
```
    cd /etc/php5/fpm/conf.d/
    
    vim 20-vld.ini
    
    	extension=vld.so
    	
    /etc/init.d/php5-fpm restart
```

这里是 `20` 指的是优先级。`extension=vld.so`，指的是 PHP 会从它的默认存放扩展库路径下去加载 `vld.so` 这个链接库文件，当然，也可以使用绝对路径的形式加载不位于 PHP 默认存放扩展库的路径下的扩展。

**注意：** 由于不是对 Nginx 服务器的修改，所以执行 `nginx -s reload` 不起作用，因为 `Nginx` 是通过 `FPM` 去处理 `PHP` 请求的。

- Apache

```
    cd /path/to/httpd/conf/
    cp /etc/php5/fpm/conf.d/20-vld.ini ./
    
    /etc/init.d/apache2 restart
    
    # Or CentOS
    service httpd restart
```

> suggestion：
> 
> - Kali 安装好之后的路径在：/usr/lib/php5/20131226。
> - ps aux | grep nginx
> - find / -name vld.so

### CLI 下简单使用

```
    # 执行该 PHP 代码 并输出的中间代码
    php -dvld.active=1 test.php
    
    # 只看输出的中间代码而不执行该 PHP 代码
    php -dvld.active=1 -dvld.execute=0 test.php
    
    # 输出的详细程度（最大为 3）
    php -dvld.active=1 -dvld.verbosity=3 t.php
    
    # 将结果输出到 .dot 文件
    php -dvld.active=1 -dvld.save_dir='D:\tmp' -dvld.save_paths=1 -dvld.dump_paths=1 t.php
```

**输出说明：**

- `Branch analysis from position：`这条信息多在分析数组时使用。
- `Return found：`是否返回，这个基本上有都有。
- `filename：`分析的文件名
- `function name：`函数名，针对每个函数 VLD 都会生成一段如上的独立的信息，这里显示当前函数的名称
- `number of ops：`生成的操作数
- `compiled vars：`编译期间的变量，这些变量是在 PHP5 后添加的，它是一个缓存优化。这样的变量在 PHP 源码中以 IS_CV 标记。
- `op list：`生成的中间代码的变量列表

### Windows 安装 VLD 扩展库

- 下载

<u>http://pecl.php.net/package/vld</u>
 
 注意，要根据 `phpinfo()` 中显示的是否线程安全下载相应的位数和版本(x64/TS/5.6)，然后解压其中的 `php_vld.dll` 文件到 `/path/to/php/ext/` 下面。
 
- 修改配置文件

编辑：`/path/to/php/php.ini`。

找到 `Windows Extensions` 然后再后面添加如下语句引入 vld 扩展即可`extension=php_vld.dll`

- 重启 Apache

刷新 `phpinfo()`，然后搜索是否已经引入 vld 扩展。

> 在 Linux 中配置文件的惯用思路并不是像在 Windows 上那样直接修改主配置文件，而往往是配置独立的配置文件，甚至是单独创建简单 的配置文件。这样的好处是基本上不用去改动主要的配置文件。
> 
> 通常，Linux 中同一个配置往往拆分成多个配置文件分开管理，甚至保存在专门的文件夹下，所以不要完全照搬 Windows 的配置思路， 以官方手册为准。

## XDebug

详见官方手册：<u>https://xdebug.org/docs/all</u>。

# 其他实践

## 代码层面

- 尽可能避免在循环中做：调用函数；创建对象；查询数据库；文件 I/O 等耗时操作
- 从方便查询的角度创建数据库表 从性能的角度使用数据类型
- 优先使用单引号

## Nginx

- 对文档类型资源启用2级压缩 不要压缩图片和音视频
- 对静态资源开启缓存

## Laravel 配合七牛云

- public 下面只留 `assets/` 和 `index.php`
- 静态的 js css imags 全部放到 `assets` 方便利用七牛的**镜像存储**
- 上传的东西 全部放到 `storage` 下面 自己新建目录

## 图片

- 为不同页面需要显示的图片做一比一的缩略图
- 单页引用的图片数量较多且较大（1M为界限）时 将图片放在专门的服务器 并做 CDN 加速
- 不要滥用图片
- 图标类小图使用 CSS Sprite 以减少 HTTP 请求次数 节省带宽


## 服务器

> 对服务器等配置是灵活 从需求出发的

- 如果用户量上升明显 => 优先增加带宽
- 如果服务器软件完成的事情越多 要求越高 比如压缩等级提升 => 优先升级 CPU
- 如果服务端程序功能复杂度提升 => 优先增加内存

## 请求

- 合并请求：减少请求数，适用于请求数可能会超过浏览器限制的情景，类似：

```
    // 前端资源请求合并
    http://example.com/?jses=a.js,b.js,c.js,d.js,e.js
    
    // 后端数据接口请求合并
    http://api.com?click=/click?k=1&b=2&answer=/answer/count?id=5&topic=/topic/5&tags=/tag?tid=10&user=/u/5

```

## 其他经验

- 没有用的全部删掉
- 在针对资源（文件／网络／数据库等）的操作时，一定要进行异常处理。

# 参考

- [VLD扩展使用指南](http://www.phppan.com/2011/05/vld-extension/)
- [深入理解PHP代码的执行的过程](https://www.2cto.com/kf/201404/290863.html)

