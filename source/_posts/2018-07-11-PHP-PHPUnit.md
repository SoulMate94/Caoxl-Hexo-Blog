---
title: 单元测试「PHPUnit」的使用
date: 2018-07-11 16:17:45
categories: PHP
tags: [PHP, PHPUnit]
---

> 本文是在`Laravel`框架下使用`PHPUnit`进行单元测试

<!-- more -->

# 为什么要单元测试 ？

> 只要你想到输入一些东西到`print`语句或调试表达式中，就用测试代替它。

# 安装`PHPUnit`

- 使用 `composer` 方式安装 `PHPUnit`

```
    composer require --dev phpunit/phpunit
```

- 安装 `Monolog`日志包, 做`phpunit`测试记录日志使用

```
    composer require monolog/monolog
```

# `PHPUnit`简单用法

## 单个文件测试

创建目录 `tests`, 新建文件 `StackTest.php`, 编辑如下:

> 一般初始化`Laravel`框架就带有`tests`文件夹, 用于存放测试代码

```php
    <?php
    
    namespace App\tests;
    
    require_once __DIR__ . '/../vendor/autoload.php';
    
    define("ROOT_PATH", dirname(__DIR__) . "StackTest.php/");
    
    use Monolog\Logger;
    use Monolog\Handler\StreamHandler;
    use PHPUnit\Framework\TestCase;
    
    class StackTest extends TestCase
    {
        public function testPushAndPop()
        {
            $stack = [];
    
            $this->assertEquals(0, count($stack));
    
            array_push($stack, 'foo');
    
            // 添加日志文件, 如果没有安装monolog，则有关monolog的代码都可以注释掉
            $this->Log()->error('hello', $stack);
    
            $this->assertEquals('foo', $stack[count($stack) - 1]);
            $this->assertEquals(1, count($stack));
    
            $this->assertEquals('foo', array_pop($stack));
            $this->assertEquals(0, count($stack));
        }
    
        public function Log()
        {
            // create a log channel
            $log = new Logger('Tester');
            $log->pushHandler(new StreamHandler(ROOT_PATH . 'storage/logs/app.log', Logger::WARNING));
            $log->error('Error');
    
            return $log;
        }
    }
```

**代码解释:**

- 1. `StackTest`为测试类
- 2. `StackTest` 继承于 `PHPUnit\Framework\TestCase`
- 3. 测试方法`testPushAndPop()`, 测试方法必须为`public`权限, 一般以`test`开头,或者你也可以选择给其加注释`@test`来表示
- 4. 在测试方法内，类似于 `assertEquals()` 这样的断言方法**用来对实际值与预期值的匹配做出断言**。

### 命令行执行

```
    phpunit命令 测试文件命名
```

例如:
```
    ./vendor/bin/phpunit tests/StackTest.php
    
    // 或者可以省略文件后缀名
    ./vendor/bin/phpunit tests/StackTest
```

执行结果:

```
    $ ./vendor/bin/phpunit tests/StackTest.php
    PHPUnit 7.2.6 by Sebastian Bergmann and contributors.
    
    .                                                                   1 / 1 (100%)
    
    Time: 203 ms, Memory: 4.00MB
    
    OK (1 test, 5 assertions)
```

我们可以在`app.log`文件中查看我们打印的日志信息。

- `storage/logs/app.log`

```
    [2018-07-11 08:45:24] Tester.ERROR: Error [] []
    [2018-07-11 08:45:24] Tester.ERROR: hello ["foo"] []
```

## 类文件引入

- `Calculator.php`

```php
    <?php
    
    class Calculator
    {
        public function sum($a, $b)
        {
            return $a + $b;
        }
    }
```

- 单元测试类: `CalculatorTest.php`

```
    <?php
    
    namespace App\tests;
    
    require_once __DIR__ . '/../vendor/autoload.php';
    require "Calculator.php";
    
    use PHPUnit\Framework\TestCase;
    
    class CalculatorTest extends TestCase
    {
        public function testSum()
        {
            $obj = new \Calculator;
    
            $this->assertEquals(2, $obj->sum(1, 1));
        }
    }
```

命令执行:

```
    $ ./vendor/bin/phpunit tests/CalculatorTest
    PHPUnit 7.2.6 by Sebastian Bergmann and contributors.
    
    .                                                                   1 / 1 (100%)
    
    Time: 185 ms, Memory: 4.00MB
    
    OK (1 test, 1 assertion)
```

如果我们把这里的断言故意写错，`$this->assertEquals(1, $obj->sum(0, 0));`

```
    $ ./vendor/bin/phpunit tests/CalculatorTest
    PHPUnit 7.2.6 by Sebastian Bergmann and contributors.
    
    F                                                                   1 / 1 (100%)
    
    Time: 194 ms, Memory: 4.00MB
    
    There was 1 failure:
    
    1) App\tests\CalculatorTest::testSum
    Failed asserting that 2 matches expected 1.
    
    C:\WWW\Laravel\tests\CalculatorTest.php:16
    
    FAILURES!
    Tests: 1, Assertions: 1, Failures: 1.
```

**会直接报出方法错误信息及行号，有助于我们快速找出`bug`**

## 高级用法

你是否已经厌烦了在每一个测试方法命名前面加一个test，是否因为只是调用的参数不同，却要写多个测试用例而纠结？
我最喜欢的高级功能，现在隆重推荐给你，叫做**框架生成器**。

- `Calculator.php`

```php
    <?php  
    
    class Calculator  
    {  
        public function sum($a, $b)  
        {  
            return $a + $b;  
        }  
    }  
    ?>
```

- 命令行启动测试用例，使用关键字 `--skeleton`

```
    ./vendor/bin/phpunit --skeleton Calculator.php
```

- 执行结果:

```
    PHPUnit 7.2.6 by Sebastian Bergmann and contributors.
    
    Wrote test class skeleton for Calculator to CalculatorTest.php.  
```

是不是很简单，因为没有测试数据，所以这里加测试数据，然后重新执行上边的命令

```php
    <?php  
    
    class Calculator  
    {  
        /** 
         * @assert (0, 0) == 0 
         * @assert (0, 1) == 1 
         * @assert (1, 0) == 1 
         * @assert (1, 1) == 2 
         */  
        public function sum($a, $b)  
        {  
            return $a + $b;  
        }  
    }  
    ?>  
```

- 原始类中的每个方法都进行`@assert`注解的检测。这些被转变为测试代码，像这样:

```
    /**
     * Generated from @assert (0, 0) == 0.
     */
    public function testSum() {
        $obj = new Calculator;
        $this->assertEquals(0, $obj->sum(0, 0));
    }
```

### 出现错误: `unrecognized option --skeleton`

```
    wget https://phar.phpunit.de/phpunit-skelgen.phar
    php phpunit-skelgen.phar
```

# 参考

- [PHPUnit中文网](http://www.phpunit.cn/)
- [PHPUnit中文文档](http://www.phpunit.cn/manual/current/zh_cn/installation.html)
- [PHP单元测试框架PHPUnit的使用](https://segmentfault.com/a/1190000011499301)