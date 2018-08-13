---
title: PHP 基础知识
date: 2017-12-31 16:03:55
categories: PHP
tags: PHP
---


> 记录一些PHP基础知识


<!--more-->


## 语法

### 流程控制

#### break
后面不跟数字，即默认情况，表示结束「本层」循环，跟数字则表示结束最近几「层」循环。

若其后所跟数字大于当前循环次数时，会出现：fatal error。

除了跳出循环，break 还可以跳出 swtich（条件判断）。

#### continue
后面不跟数字，即默认情况，表示结束「本次」循环，跟数字则表示结束最近几「次」循环。

#### goto

快速跳出多重循环和多重分支。

只能在同一个文件和作用域中跳转，也就是说，goto 无法跳出本函数或者方法，也无法跳入另一个函数。

goto 只能在 php > 5.3 种使用。

### 变量

#### 全局变量
在整个作用域（整个 PHP 文件中）都是可见的。

#### 引号中的变量
为确保双引号中的变量被正确解析，可以用大括号 {} 包裹起来，比如："{$v}"。

#### 超全局变量
也叫「预定义超全局数组」。

通过预定义超全局数组，我们可以获取程序中需要的各种数据和环境相关的参数值 。

超全局变量除了具有全局变量的特点外，还具有不需要声明和定义（声明也不会报错但无意义），可直接访问的特点。

意味着在一个脚本的全部作用域都可用，本质是 PHP 引擎为了方便、快捷编程，而事先封装好的数据。

PHP 默认有以下 9 种预定义超全局数组：

##### $_GET
存储通过 URL 参数传递给当前脚本的数组变量。有 3 种使用方式：
 - 超链接
 - 表单
 - `header()` 跳转中带参数提交：`header('Location: query.php?param=val')`;

##### 关于 register_globals 引起的 SQL 注入

这个特性（缺陷）在 php >= 5.4 后已经被移除了，在低于 5.4 的 php.ini 文件中，确保其值为 off。

其原本带来的隐患是，从 `$_GET`/`$_POST`/`$_REQUEST` 中接收到的参数，到接受的 PHP 脚本后，可以自动通过和参数同名的变量访问，并且能覆盖 PHP 文件中没有声明过的变量。比如 `$_GET['foo']` 可以通过 `$foo` 来访问其值。

##### $_POST
上传文件必须使用 POST 方式。和 GET 方式相比，优点是更安全和传输的数据大小不受 HTTP 限制（都会受浏览器限制）。

但是，收藏书签时，GET 比 POST 更方便。


##### $_REQUEST
包含了 `$_GET`/`$_POST`/`$_COOKIE`。常用于不清楚提交方式时接受参数。

不过因为包含了三种方式，可能会受到三种方式缺点的攻击，所以并不可信。

判断是 GET 还是 POST，可以通过 `$_SERVER['REQUEST_METHOD']` 来获得。

##### $_SERVER
主要包含客户端发来的 HTTP 请求行和消息头的信息，同时还有服务器自己的信息，比如 `Document Root` 和 `Query String` 等。


##### $_ENV
该全局数组可以获取服务器环境变量，在 PHP5.3 中默认禁用，但是可以手动启动：

php.ini => `variables_order = 'EGPCS'` 。

生产环境，最好不要启用。


##### $GLOBALS
注意这个变量名没有下划线 `_`。

它是一个包含了全部全局变量的全局组合数组，全局变量的名字就是该数组的键。

同时一个自定义的全局变量, 也会自动地被 `$GLOBALS` 管理。

##### $_FILE
文件上传数组，包含了上传文件的所需数据。

##### $_COOKIE

**关于「会话」**

打开浏览器访问一个网站的很多页面后，只有当关闭浏览器的时候，和该网站的一次会话才算结束

cookie 属于客户端（浏览器）技术，是服务器／网站保存在浏览器上的信息，供服务器在需要的时候使用。

其特点是：以 `***.txt` 保存，不能保存太多的信息（顾名思义：小甜饼）。

**cookie 基本使用：**
```
    # 创建/更新 cookie
    setcookie($key1, $val1, $expire1);    // $expire 是过期的具体时间点对应的时间戳
    setcookie($key2, $val2, $expire2);
    // ...
    # 获得 cookie
    $cookie = $_COOKIE['key'];
    # 删除 cookie
    setcookie($key, null, time()-1024);    // 清除某条 cookie
    $_COOKIE = [];    // 清除所有 cookie
```

如果不指定失效时间，则 cookie 不会保存到本地客户端，当浏览器的会话结束这个 cookie 就会失效。

一般来说，cookie 不用于保存重要信息。

cookie 保存中文时候，默认会将中文进行 urlencode 编码。

浏览器在访问网站的时候会把 cookie 信息发送给网站的服务器（如果有的话）。

**cookie 能保存的只能是字符串，不能是对象。**

和 cookie 一样，也是属于会话技术，只是 session 文件（也可以存到缓存或数据库）通常保存在服务器。一般情况，一个 会话就对应一个 session 文件。

服务器在运行时可以为每个用户创建一个 session 文件，当用户再去访问同一台服务器中的其他 web 资源时，可以取出各自的 session 数据为每个用户提供不同的服务。

session 保存格式举例：`key | s : size`。其中，`key` 是键值；`s` 是数据类型，可以为 `double ,integer, bool, array, object；size` 代表元素的个数或者数据的字节单位大小。

session 在很大程度上弥补了 cookie 的带宽和安全性问题，而两者基本上都是一起配合使用的。

**session 基本使用：**
```
    # 初始化 session
    session_start();    // 只要是需要使用 session 都需要先初始化
    // 也可以不用每次都初始化 php.ini => session.auto_start = 1;
    // 不推荐因为影响效率, 因为不是每个网站都需要 session
    # 新增 session 纪录
    $_SESSION[$key] = $val;
    # 获取 session
    print_r($_SESSION);    // 打印所有 session
    $val = $_SESSION[$key];    // 获得 session 中 key 为 $key 的值
    echo $val;    // $val 为基本数据类型
    print_r($val[$index]);    // $val 为数组
    print_r($val->getName());    // $val 为对象
    # 更新 session
    // 思路同 cookie 重置即可
    # 删除 session
    unset($_SESSION[$key]);     // 删除 session 中 key 值为 $key 的纪录
    session_destroy();    // 删除当前浏览器对应的 session 文件
```

默认存活时间 24 min，使用了 session 文件则存活时间重新计时 ，可以修改配置：`php.ini => session:gc_maxlifetime = 1440`。（也称发呆时间，不同 PHP 版本中这个配置有可能不一样）

**session 使用有关问题：**
 - 服务器是如何实现一个 session 为一次会话服务的？
 通过客户端传来的 sessid，可以从 cookie 中获得 sessid，也可以从 URL 参数中获得。
 - 用户禁用 cookie 之后, 服务器每次 session_start() 都会创建一个全新的 session 文件，就无法让多个 PHP 去共享一份 session 文件，此时如何使用 session？
 可以在多个 PHP 文件之间传递 session id，然后通过 session id 来读取共享数据。
 ```
    # 1. 通过超链接／GET 请求
    if (isset($_GET['PHPSESSID']))
      session_id($_GET['PHPSESSID']);    // 设置当前的 session id
    session_start();
    // session_id();    // 查看当前的 session id
    # 使用指定 session id 的 session
    print_r($_SESSION['key']);
    # 2. 通过常量 SID
    // 将 ".SID." 插入到超链接中
    // php.ini => session.use_trans_sid(透明 SID 支持) 置1, 重启 Apache/FPM
 ```
 
 - 自定义 session 处理器
 通过 `session_set_save_handler` 来实现。需要更改配置文件：`session.save_handler = files` 改为 `session.save_handler = user`。
 
 - session 可以存放的位置
 文件，数据库，内存，网络文件系统（nfs，也就是另一台服务器）。
 
 - 关于 session 的垃圾回收机制
 ```
    以下三个文件的配置共同决定了session的垃圾回收机制(gc)
    session.gc_maxlifetime = 1440    // 发呆时间
    session.gc_probability = 1       // gc 触发概率分子
    session.gc_divisor = 1000        // gc 触发概率分母
 ```
 
 说明：网站规模越大，`session.gc_probability` / `session.gc_divisor` 值应该越小，可以减少服务器压力。
 
 ```
    session.use_cookie = 1;     // 是否启用 cookie
    session.cookie_path =/      // cookie 有效的路径
    session.domain_name =       // 根据规范产生域名, 自动生成, 无需设置
    session.cookie_lifetime = 0 // cookie 保存时间, 默认 0, 表示关闭浏览器就失效
 ```
 
### 数组
数组就是连续存储数据的数据结构。

PHP 有两种数组：索引数组和关联数组，索引和关联两个词都是针对数组的键而言的。

#### 索引数组
索引数组是指数组的键是整数的数组，并且键的整数顺序是从0开始，依次类推。

##### 索引数组赋值有三种方式
 - 第一种：用数组变量的名字后面跟一个中括号的方式赋值，当然，索引数组中，中括号内的键一定是整 数。比如，`$arr[0]='苹果'`;
 - 第二种：用 array() 创建一个空数组，使用 => 符号来分隔键和值，左侧表示键，右侧表示值。当然，索引数组中，键一定是整数。比如，`array('0'=>'苹果')`;
 - 第三种：用 array() 创建一个空数组，直接在数组里用英文的单引号 ' 或者英文的双引号 " 赋值，数组会默认建立从0开始的整数的键。比如 `array('苹果')`; 这个数组相当于 `array('0'=>'苹果')`;
 
##### 访问索引数组内容

用数组变量的名字后跟的中括号中的键，来访问数组中的值。例如：

```
    $fruit  = array('苹果','香蕉');
    $fruit0 = $fruit['0'];
    print_r($fruit0);    // 结果为苹果
```

#### 关联数组

关联数组是指数组的键是字符串的数组。

```
    $fruit = array(
      'apple'     => "苹果",
      'banana'    => "香蕉",
      'pineapple' => "菠萝"
    );
```
 
##### 关联数组赋值
关联数组赋值有两种方式:
 
 - 第一种：用数组变量的名字后面跟一个中括号的方式赋值，当然，关联数组中，中括号内的键一定是字符串。比如，`$arr['apple']='苹果'`;
 - 第二种：用 array() 创建一个空数组，使用 => 符号来分隔键和值，左侧表示键，右侧表示值。当然，关联数组中，键一定是字符串。比如，`array('apple'=>'苹果')`;
 
##### 访问关联数组内容
用数组变量的名字后跟中 括号+键的 方式来访问数组中的值，键使用单引号或者双引号括起来

```
    $fruit  = array(
      'apple'     => "苹果",
      'banana'    => "香蕉",
      'pineapple' => "菠萝"
    );
    $fruit0 = $fruit['banana'];
    print_r($fruit0);
```

#### 当小数作为数组下标时会怎样？

当小数作为下标时，自动截取整数部分作为下标。

#### 数组越界问题

当访问到越界元素时，PHP 会提示但不会报错，PHP中的数组可以动态增长的数组 。

### 语言结构

#### require 和 include

 - require
 节省资源、避免重复定义的错误 。
 
 - include
 include 需要用到 php 文件时才引入，且出现错误会继续执行， 而 require 遇到错误会终止程序
 
 - require_once 和 include_once
 `_once` 表示如果被包含文件已经包含进去了则不再包含。
 
 - 结论
 项目中最好使用 require_once，但是 php 页面中也不可能只用 require_once，因为有些页面内容确实需要调用多次。
 
### 错误和异常

#### 手动触发错误
```
    trigger_error('错误提示', E_USER_WARNING);
```

#### 抛出一个异常
从PHP5开始，PHP 支持异常处理，异常处理是面向对象一个重要特性，PHP 代码中的异常通过 throw 抛出，异常抛出之后，后面的代码将不会再被执行。

既然抛出异常会中断程序执行，那么为什么还需要使用异常处理呢？

异常抛出被用于在遇到未知错误，或者不符合预先设定的条件时，通知客户程序，以便进行其他相关处理，不至于使程序直接报错中断。当代码中使用了 try catch 的时候，抛出的异常会在 catch 中捕获，否则会直接中断。

```
    try{
    	// 可能出现错误或异常的代码
    } catch (Exception $e) {
    	// 对异常处理
      	// 1、自己处理
      	// 2、不处理，将其再次抛出
    }
```

使用异常的函数应该位于 `try{}` 代码块内。如果没有触发异常，则代码将照常继续执行，但是如果异常被触发，会抛出一个异常。

`catch {}` 代码块会捕获异常，并创建一个包含异常信息的对象。`（Exception 是 php 已定义好的异常类）`

另外，`throw` 规定如何触发异常。**每一个 throw 必须对应至少一个 catch**，当然可以对应多个。
```
    // 检测数字是否大于 1, 如果是, 则抛出一个异常。
    function checkNum($number)
    {
    	if ($number > 1) {
    		throw new Exception('数字必须小于等于 1');
        }
    	return true;
    }
    // 在 try 代码块中触发异常
    try {
    	checkNum(2); // 如果异常被抛出，那么下面一行代码将不会被输出
      	echo '如果能看到这个提示，说明你的数字小于等于 1';
    } catch(Exception $e){
    	// 捕获异常
    	echo '捕获异常: ', $e->getMessage();
    }
```

执行上面代码将获得类似这样一个错误: *‘捕获异常: 数字必须小于等于 1’*。

其实上面的代码演示的就是如何抛出一个异常，并捕获它的过程：

在 `try {}` 代码块中调用 `checkNum()` 函数。执行 `checkNum(2)` 将触发异常并抛出一个 `Exception` 异常对象。由于 `catch {}` 代码块中事先已经准备好接收捕获该异常类对象，因此此时会创建一个包含该异常信息的对象 $e。 通过从这个 `Exception` 对象调用 `$e->getMessage()`，输出来自该异常的错误消息文本。

#### 异常处理类

PHP 具有很多异常处理类，其中 Exception 是所有异常处理的基类，具有几个基本属性与方法：
 - message：异常消息内容
 - code：异常代码
 - file：抛出异常的文件名
 - line：抛出异常在该文件的行数
 
具有的常用方法有：
 - getTrace：获取异常追踪信息
 - getTraceAsString：获取异常追踪信息的字符串
 - getMessage：获取出错信息
 
如果必要的话，可以通过继承 Exception 类来建立自定义的异常处理类。

```
    // 自定义的异常类，继承了 PHP 的异常基类 Exception
    class MyException extends Exception
    {
    	public function getInfo() {
    		return '自定义错误信息';
    	}
    }
    try {
        // 使用异常的函数应该位于 `try` 代码块内
        // 如果没有触发异常，则代码将照常继续执行
        // 但是如果异常被触发，会抛出一个异常
    	throw new MyException('error');
    } catch(Exception $e) {
    	// `catch` 代码块内捕获异常，并创建一个包含异常信息的对象
    	echo $e->getInfo();    // 获取自定义的异常信息
    	echo $e->getMessage(); // 获取继承自基类的 getMessage 信息
    }
```
 
#### 捕获异常信息

在了解了异常处理的基本原理之后，我们可以通过 `try...catch...` 来捕获异常，我们将执行的代码放在 `try` 代码块中，一旦其中的代码抛出异常，就能在 `catch` 中捕获。

这里我们只是通过案例来了解 `try catch` 的机制以及异常捕获的方法，在实际应用中，不会轻易的抛出异常，只有在极端情况或者非常重要的情况下，才会抛出异常。

抛出异常，可以保障程序的正确性与安全，避免导致不可预知的 `bug`。

一般的异常处理流程代码为：
```
    try {
    	throw new Exception('wrong');
    } catch(Exception $e) {
      echo 'Error:', $e->getMessage(), PHP_EOL;
      echo $e->getTraceAsString(), PHP_EOL;
    }
    echo '异常处理后，继续执行其他代码';
```

#### 获取错误发生的所在行
在异常被捕获之后，我们可以通过异常处理对象获取其中的异常信息，前面我们已经了解
捕获方式，以及获取基本的错误信息。

在实际应用中，我们通常会获取足够多的异常信息，然后写入到错误日志中。

通过我们需要将报错的文件名、行号、错误信息、异常追踪信息等记录到日志中，以便调
试与修复问题。

### 字符串

#### 字符串定义方式
一个字符串通过下面的 3 种方法来定义：
 - 单引号
 - 双引号
 - heredoc语法结构

#### 单引号和双引号区别

PHP 允许我们在双引号串中直接包含字串变量，而单引号串中的内容总被认为是普通字符。

#### 去除字符串首尾的空格

PHP 中有 3 个函数可以去掉字符串的空格
 - trim：去除一个字符串两端空格。
 - rtrim：去除一个字符串右部空格，其中的 r 是 right 的缩写。
 - ltrim：去除一个字符串左部空格，其中的 l 是 left 的缩写。
 
 ```
    echo trim(" 空格 "), PHP_EOL;
    echo rtrim(" 空格 "), PHP_EOL;
    echo ltrim(" 空格 "), PHP_EOL;
 ```
 
#### 获取字符串的长度

strlen 函数对于计算英文字符是非常的擅长，但是如果有中文汉字，要计算长度该怎么办?

可以使用 mb_strlen() 函数获取字符串中中文长度。

```
    $str = '我爱你';
    echo mb_strlen($str, 'UTF8'); // 结果为 3
```

#### 字符串的截取

- 英文字符串的截取函数 substr()

```
    // 函数说明: substr(字符串变量, 开始截取的位置, 截取个数)
    $str = 'i love you';
    // 截取 love 这几个字母
    echo substr($str, 2, 4);
```

为什么开始位置是 2 呢？因为 substr 函数计算字符串位置是从 0 开始的。

- 中文字符串的截取函数 mb_substr()

```
    // 函数说明: mb_substr(字符串变量, 开始截取的位置, 截取个数, 网页编码)
    $str='我爱你，中国';
    // 截取中国两个字
    echo mb_substr($str, 4, 2, 'utf8');
```

#### 查找字符串

函数说明：`strpos(要处理的字符串, 要定位的字符串, 定位的起始位置[可选])`

```
    $str = 'I want to study at home';
    $pos = strpos($str, 'home');
    echo $pos;    // 结果显示19，表示从位置0开始，`home` 在第19个位置开始出现
```

#### 替换字符串

函数说明: `str_replace(要查找的字符串, 要替换的字符串, 被搜索的字符串, 替换进行计数[可选])`

```
    $str = 'I want to learn js';
    $replace = str_replace('js', 'php', $str);
    echo $replace;    // 结果: I want to learn php
```

#### 格式化字符串

函数说明: `sprintf(格式, 要转化的字符串)`

```
    $str = '99.9';
    $result = sprintf('%01.2f', $str);
    echo $result;    // 结果显示 99.90
```

**解释下，这个例子中的格式：**
 - `%` 是开始的意思，写在最前面表示指定格式开始了。也就是 “起始字符”，直到出现 “转换字符” 为止，就算格式终止。
 - 跟在 `%` 符号后面的是 `0`， 是 `“填空字元”` ，表示如果位置空着就用 0 来填满。
 - 在 `0` 后面的是 `1`，这个 `1` 是规定整个所有的字符串占位要有 1 位以上( 小数点也算一个占位 )。如果把 `1` 改成 `6`，则 `$result` 的值将为 `099.90`
   因为，在小数点后面必须是两位，`99.90` 一共 `5` 个占位，现在需要 `6` 个占位，所以用 0 来填满。
 - 在 `%01` 后面的 `.2` 就很好理解了，它的意思是，**小数点后的数字必须占2位**。 如果这时候，$str 的值为 9.234,则 $result 的值将为 9.23.
 为什么 4 不见了呢? 因为在小数点后面，按照上面的规定，必须且仅能占 2 位。
 可是 $str 的值 中，小数点后面占了3位，所以，尾数 4 被去掉了，只剩下 23。
 - 最后，以 `f “转换字符”` 结尾。
 
#### 字符串的合并与分割
 - 字符串合并：implode()
 ```
    // 函数说明: implode(分隔符[可选], 数组)
    $arr = array('Hello', 'World!');
    $result = implode(' ', $arr);
    print_r($result);   // 结果显示: Hello World!
 ```
 
 - 字符串分隔：explode()
 ```
    // 函数说明: explode(分隔符[可选], 字符串)
    $str = 'apple,banana';
    $result = explode(',', $str); print_r($result);//结果显示array('apple','banana')
 ```
 
 - 字符串的转义
 addslashes() 函数用于对特殊字符加上转义字符。
 ```
    $str = "what's your name?";
    echo addslashes($str);    // 输出: what\'s your name?
 ```

### 函数

#### 返回值

 - 使用 `return` 关键字可以使函数返回值，可以返回包括数组和对象的任意类型。
 - 如果省略了 `return`，则默认返回值为 `NULL`。
 - 函数不能返回多个值，但可以通过返回一个数组来得到类似的效果。
 ```
    function getNums() {
      return [1, 2, 3];
    }
    list($one, $two, $three) = getNums();
 ```

#### 可变函数

所谓可变函数，即通过变量的值来调用函数，因为变量的值是可变的，所以可以通过改变一个变量的值来实现调用不同的函数。

可变函数经常会用在回调函数、函数列表，或者根据动态参数来调用不同的函数。可变函数的调用方法为变量名加括号。

```
    function name () {
      return 'caoxl';
    }
    $fun = 'name';
    $fun();    // 调用可变函数
```

可变函数也可以用在对象的方法调用上：

```
    class Book
    {
    	publich function getName()
        {
       		return 'Modern PHP';
    	}
    }
    $fun  = 'getName';
    $book = new Book();
    $booo->$fun();
```

#### 内置函数

内置函数指的是 PHP 默认支持的函数，PHP内置了很多标准的常用的处理函数，包括字符串处理、数组函数、文件处理、session 与 cookie 处理等。

我们先拿字符串处理函数来举例，通过内置函数 `str_replace `可以实现字符串的替换。

下面的例子将 “jobs” 替换成 “steven jobs”：
```
    $str = 'i am jobs.';
    $str = str_replace('jobs', 'steven jobs', $str);
    echo $str;
```

另外，一些函数是通过其他扩展来支持的。比如：操作 mysql 数据库的函数是通过 mysqli 扩展来支持的。


 
## 会话控制

### Cookie

Cookie 是存储在客户端浏览器中的数据，我们通过 Cookie 来跟踪与存储用户数据。一般情况下，Cookie 通过 HTTP headers 从服务端返回到客户端。多数 web 程序都支持 Cookie 的操作，因为 Cookie 是存在于 HTTP 的标头之中，所以必须在其他信息输出以前进行设置，类似于 header 函数的使用限制。

PHP 通过 `setcookie()` 函数进行 Cookie 的设置，任何从浏览器发回的 Cookie，PHP 都会自动的将他存储在 `$_COOKIE` 的全局变量之中，因此我们可以通过 `$_COOKIE['key']` 的形式来读取某个 `Cookie` 值。

PHP 中的 Cookie 具有非常广泛的使用，经常用来存储用户的登录信息， 购物车等，且在使用会话 Session 时通常使用 Cookie 来存储会话 id 来识别用户，Cookie 具备有效期，当有效期结束之后，Cookie 会自动的从 客户端删除。同时为了进行安全控制，Cookie 还可以设置域跟路径。

#### 设置 Cookie

##### setcookie

PHP设置 Cookie 最常用的方法就是使用 setcookie 函数。setcookie 具有 7 个可选参数，我们常用到的为前5个：

 - **name：** Cookie名，可以通过 $_COOKIE['name'] 进行访问。
 - **value：** Cookie的值。
 - **expire：** 过期时间，Unix 时间戳格式，默认为 0，表示浏览器关闭即失效。
 - **path：** 有效路径。如果路径设置为 /，则整个网站都有效。
 - **domain：** 有效域。默认整个域名都有效，如果设置了 `www.myapp.com’，则只在 www 子域中有效
 
 ```
    $value = 'test';
    setcookie("TestCookie", $value);
    setcookie("TestCookie", $value, time()+3600); // 有效期一小时
    setcookie("TestCookie", $value, time()+3600, "/path/", "myapp.com"); // 设置路径与域
 ```

##### setrawcookie

PHP 中还有一个设置 Cookie 的函数 setrawcookie。

setrawcookie 跟 setcookie 基本一样，唯一的不同就是 value 值不会自动的进行 urlencode，因此在需要的时候要手动的进行 urlencode。

```
    setrawcookie('cookie_name', rawurlencode($value), time()+606024*365);
```

因为 Cookie 是通过 HTTP 标头进行设置的，所以也可以直接使用 header 方法进行设置。

```
    header('Set-Cookie:key=value');
```

##### cookie 的删除与过期时间

在 PHP 中删除 cookie 也是采用 setcookie 函数来实现的，只需要设置下失效时间即可：

```
    setcookie('test', '', time()-1);
```

可以看到将 cookie 的过期时间设置到当前时间之前，则该 cookie 会自动失效，也就达到了删除 cookie 的目的。

之所以这么设计是因为 cookie 是通过 HTTP 的标头来传递的，客户端根据服务端返回的 Set-Cookie 段来进 行 cookie 的设置，如果删除 cookie 需要使用新的 Del-Cookie 来实现， 则 HTTP 头就会变得复杂，实际上仅通过 Set-Cookie 就可以简单明了的实现 Cookie 的设置、更新与删除。

因此，也可以直接通过 header 函数来删除 cookie。

```
    header(
    
      'Set-Cookie: test=1393832059; expires='.gmdate(
      
        'D, d M Y H:i:s \G\M\T',
        
    	time()-1
    	
      )
      
    );
```
这里用到了`gmdate()`，用来生成格林威治标准时间，以便排除时差的影响。

##### cookie 的有效路径

cookie 中的路径用来控制设置的 cookie 在哪个路径下有效，默认为 `’/‘`， 在所有路径下都有。

当设定了其他路径之后，则只在设定的路径以及子路径下有效，例如：

```
    setcookie('test', time(), 0, '/path');
```

上面的设置会使 test 在 `/path` 以及子路径 `/path/abc` 下都有效，但是在根目录下就读取不到 test 的 cookie 值。

一般情况下，大多是使用所有路径的，只有在极少数有特殊需求的时 候，会设置路径，这种情况下只在指定的路径中才会传递 cookie 值，可以节省数据的传输，增强安全性以及提高性能。

当我们设置了有效路径的时候，不在当前路径的时候则看不到当前cookie。

```
    setcookie('test', '1',0, '/path');
    
    var_dump($_COOKIE['test']);
```

### Session

#### Cookie 的局限

cookie 将数据存储在客户端，建立起用户与服务器之间的联系，通常可以解决很多问题，但是 cookie 仍然具有一些局限：
 - cookie 相对不是太安全，容易被盗用导致 cookie 欺骗
 - 单个 cookie 的值最大只能存储 4k
 - 每次请求都要进行网络传输，占用带宽

#### Session 的优点

session 是将用户的会话数据存储在服务端，没有大小限制，只通过一个 session id 进行用户识别


#### Session 和 Cookie 的关系

默认情况下 PHP 里的 session id 是通过 cookie 来保存的，因此从某种程度上来说，seesion 依赖于cookie。但这不是绝对的，session id 也可以通过参数来实现，**只要能将 session id 传递到服务端进行识别的机制都可以使用 session。**

#### 使用 Session

在 PHP 中使用 session 非常简单，先执行 `session_start()` 方法开启 session， 然后通过全局变量$_SESSION 进行 `session` 的读写。

```
    session_start();
    
    $_SESSION['test'] = time();
    
    print_r($_SESSION);
```

session 会自动的对要设置的值进行 encode 与 decode，因此 session 可以支持任意数据类型，包括数据与对象等。

```
    session_start();
    
    $_SESSION['ary'] = array('name' => 'jobs');
    
    $_SESSION['obj'] = new stdClass();
    
    print_r($_SESSION);
```

默认情况下，session 是以文件形式存储在服务器上的，因此当一个页面开启了 session 之后，会独占这个session 文件，这样会导致当前用户的其他并发访问无法执行而等待。

可以采用缓存或者数据库的形式存储来解决这个问题。

#### 删除与销毁 Session

删除某个 session 值可以使用 PHP 的 unset 函数，删除后就会从全局变量 `$_SESSION` 中去除，无法访问。

```
    session_start();
    
    $_SESSION['name'] = 'jobs';
    
    unset($_SESSION['name']);
    
    echo $SESSION['name'];    // 提示name不存在
```

如果要删除所有的 session，可以使用 `session_destroy 函数`销毁当前 session。session_destroy 会删除所有数据，但是 session_id 仍然存在。
```
    session_start();
    
    $_SESSION['name'] = 'jobs';
    $_SESSION['time'] = time();
    
    session_destroy();
```

值得注意的是，`session_destroy`并不会立即的销毁全局变量 `$_SESSION` 中的值，只有当下次再访问的时候，`$_SESSION` 才为空。因此如果需要立即销毁 `$_SESSION`，可以使用 unset 函数。

```
    session_start();
    
    $_SESSION['name'] = 'jobs';
    $_SESSION['time'] = time();
    
    unset($SESSION);
    
    session_destroy();
    var_dump($_SESSION);     // 此时已为空
```

如果需要同时销毁 cookie 中的 session_id，比如在用户退出的时候可能会用到，则还需要显式的调用setcookie 方法删除 session_id 的 cookie 值。

#### 使用 session 来存储登录用户

session 可以用来存储多种类型的数据，因此具有很多的用途，常用来存储用户的登录信息，购物车数据，或者一些临时使用的暂存数据等。

用户在登录成功以后，通常可以将用户的信息存储在 session 中，一般的会单独的将一些重要的字段单独存储，然后所有的用户信息独立存储。

```
    $_SESSION['uid'] = $userinfo['uid'];
    $_SESSION['userinfo'] = $userinfo;
```

一般来说，登录信息既可以存储在 sessioin 中，也可以存储在 cookie 中， 他们之间的差别在于 session 可以方便的存取多种数据类型，而 cookie 只支持字符串类型，同时对于一些安全性比较高的数据，cookie 需要进行格式化与加密存储，而 session 存储在服务端则安全性较高。

```
    <?php
    session_start();
    
    // 假设用户登录成功获得了以下用户数据
    $userinfo = array(
      'uid'   => 1024,
      'name'  => 'cjli',
      'email' => 'cjli@info.com',
      'sex'   => 'man',
      'age'   => '18'
    );
    
    header('Content-Type:text/html; charset=utf-8');
    
    // 将用户信息保存到 session 中
    $_SESSION['uid']      = $userinfo['uid'];
    $_SESSION['name']     = $userinfo['name'];
    $_SESSION['userinfo'] = $userinfo;
    
    // 将用户数据保存到 cookie 中的一个简单方法
    $secure_key = 'secure_key';     // 加密密钥
    $str = serialize($userinfo);    // 将用户信息序列化
    
    // 用户信息加密前
    $str = base64_encode(mcrypt_encrypt(MCRYPT_RIJNDAEL_256, md5($secure_key), $str, MCRYPT_MODE_ECB));
    
    // 用户信息加密后
    // 将加密后的用户数据存储到cookie中
    setcookie('userinfo', $str);
    
    // 当需要使用时进行解密
    $str = mcrypt_decrypt(MCRYPT_RIJNDAEL_256, md5($secure_key), base64_decode($str), MCRYPT_MODE_ECB);
    
    $uinfo = unserialize($str);
    
    echo '解密后的用户信息', PHP_EOL;
    
    print_r($uinfo);
```

## 日期时间

### 取得当前的 Unix 时间戳

UNIX 时间戳( timestamp )是 PHP 中关于时间与日期的一个很重要的概念，它表示从 1970年1月1日 00:00:00 到当前时间的秒数之和。
PHP 提供了内置函数 `time()` 来取得服务器当前时间的时间戳。
```
    echo time(); // 1396193923
```

这个数字表示从 1970年1月1日 00:00:00 到我输出这个脚本时经历了1396193923 秒。

### 取得当前的日期

php内置了 `date()` 函数，来取得当前的日期。

```
    date(时间戳的格式, 规定时间戳[默认是当前的日期和时间，可选])
```

返回值：日期和时间。比如：

```
    // 第二个参数取默认值的情况
    echo date("Y-m-d");
    
    // 第二个参数有值的情况
    echo date("Y-m-d",'1396193923');    
```

### 将格式化的日期字符串转换为 Unix 时间戳

strtotime 函数预期接受一个包含美国英语日期格式的字符串，并尝试将其解析为 Unix 时间戳。

```
    strtotime(要解析的时间字符串, 计算返回值的时间戳[默认是当前的时间，可选])
```

返回值：成功则返回时间戳，否则返回 FALSE。

```
    echo strtotime("now");    // 相当于将英文单词 now 直接等于现在的日期和时间，并把这个日期时间转化为 unix 时间戳; 这个效果跟 `echo time();` 一样
    
    echo strtotime("+1 seconds");    // 相当于将现在的日期和时间加上了 1 秒，并把这个日期时间转化为unix 时间戳。这个效果跟 `echo time()+1;` 一样
    
    echo strtotime("+1 day");    // 相当于将现在的日期和时间加上了 1 天
    
    echo strtotime("+1 week");   // 相当于将现在的日期和时间加上了 1 周
    
    echo strtotime("+1 week 3 days 7 hours 5 seconds");    // 相当于将现在的日期和时间加上了 1周3天7小时5秒
```

### 格式化格林威治(GMT)标准时间

`gmdate()` 函数能格式化一个 GMT 的日期和时间，返回的是格林威治标准时(GMT)。
举个例子，我们现在所在的中国时区是东八区，领先格林威治时间 8 个小时，有时候也叫 GMT+8，那么服务器运行以下脚本返回的时间应该是这样的:

```
    echo date('Y-m-d H:i:s', time());    // 当前时间假定是 2014-05-01 15:15:22
    echo gmdate('Y-m-d H:i:s', time());  // 输出 2014-05-01 07:15:22
```

因为格林威治时间，是现在中国时区的时间减去 8 个小时，所以相对于现在时间要少 8 个小时。


## GD库

GD 指的是 Graphic Device，PHP 的 GD 库是用来处理图形的扩展库，通过 GD 库提供的一系列 API，可以对图像进行处理或者直接生成新的图片。

PHP 除了能进行文本处理以外，通过 GD 库，可以对 JPG、PNG、GIF、SWF 等图片进行处理。

GD 库常用在图片加水印，验证码生成等方面。PHP 默认已经集成了 GD 库，只需要在安装的时候开启就行。

### 输出一张图片

```
    header('Content-Type: image/png');
    $img = imagecreatetruecolor(100, 100);
    $red = imagecolorallocate($img, 0xFF, 0x00, 0x00);
    imagefill($img, 0, 0, $red);
    imagepng($img);
    imagedestroy($img);
```

### 绘制线条

要对图形进行操作，首先要新建一个画布。

通过 imagecreatetruecolor 函数可以创建一个真彩色的空白图片：

```
    $img = imagecreatetruecolor(100, 100);
```

GD 库中对于画笔所用的颜色，需要通过 imagecolorallocate 函数进行分配，通过参数设定 RGB的颜色值来确定画笔的颜色：

```
    $red = imagecolorallocate($img, 0xFF, 0x00, 0x00);
```

然后我们通过调用绘制线段函数 imageline 进行线条的绘制，通过指定起点跟终点来最终得到线条：

```
    imageline($img, 0, 0, 100, 100, $red);
```

线条绘制好以后，通过 header 与 imagepng 进行图像的输出。

```
    header('Content-Type: image/png');
    imagepng($img);
```

最后可以调用 imagedestroy 释放该图片占用的内存。

```
    imagedestroy($img);
```

通过上面的步骤，可以发现 PHP 绘制图形非常的简单，但很多时候我们不只是需要输出图片，可能我们还需要得到一个图片文件，可以通过 imagepng 函数指定文件名将绘制后的图像保存到文件中：

```
    imagepng($img, 'img.png');
```

### 在图像中绘制文字

GD 库可以进行多种图形的基本操作，常用的有绘制线条，背景填充，画矩形，绘制文字等。

跟绘制线条类似，首先需要新建一个图片与初始化颜色。

```
    $img = imagecreatetruecolor(100, 100);
    
    $red = imagecolorallocate($img, 0xFF, 0x00, 0x00);
```

然后使用 imagestring 函数来进行文字的绘制，这个函数的参数很多：

`imagestring(resource $image, int $font, int $x, int $y, string $s, int $col)`

可以通过 `$font` 来设置字体的大小，x, y 设置文字显示的位置，`$s` 是要绘制的文字，`$col` 是文字的颜色。

```
    imagestring($img, 5, 0, 0, 'Hello world', $red);
    header('Content-Type: image/png');
    imagepng($img);
    imagedestroy($img);
```

### 输出图像文件

前面我们已经了解到，通过 imagepng 可以直接输出图像到浏览器，但是很多时候，我们希望将处理好的图像保存到文件，以便可以多次使用。

通过指定路径参数将图像保存到文件中。

```
    $filename = 'img.png';
    imagepng($img, $filename);
```

使用不同的函数保存为不同的图片格式：
 - imagepng 可以将图像保存成 png 格式
 - imagejpeg 将图片保存成 jpeg 格式
 - imagegif 将图片保存成 gif 格式
 
需要说明的是，imagejpeg 会对图片进行压缩，因此还可以设置一个质量参数。
```
    $filename = 'img.jpg';
    imagejpeg($img, $filename, 80);
```

### 生成图像验证码

简单的验证码其实就是在图片中输出了几个字符，通过 imagestring 函数就能实现。

但是在处理上，为了使验证码更加的安全，防止其他程序自动识别，因此常常需要对验证码进行一些干扰处理，通常会采用绘制一些噪点，干扰线段，对输出的字符进行倾斜、扭曲等操作。

可以使用 imagesetpixel 绘制点来实现噪点干扰，但是只绘制一个点的作用不大，因此这里常常会使用循环进行随机绘制。

```
    for($i=0; $i<50; $i++) {
      imagesetpixel($im, rand(0, 100) , rand(0, 100) , $black);
      imagesetpixel($im, rand(0, 100) , rand(0, 100) , $green);
    }
```

### 给图片添加水印

给图片添加水印的方法一般有两种，一种是在图片上面加上一个字符串，另一种是在图片上加上一个 logo 或者其他的图片。

因为这里处理的是已经存在的图片，所以可以直接从已存在的图片建立画布，通过 imagecreatefromjpeg 可以直接从图片文件创建图像。

```
    $im = imagecreatefromjpeg($filename);
```

创建图像对象以后，我们就可以通过前面的 GD 函数，绘制字符串到图像上。

如果要加的水印是一个 logo 图片，那么就需要再建立一个图像对象，然后通过 GD 函数 imagecopy

将 logo 的图像复制到源图像中。

```
    $logo = imagecreatefrompng($filename);
    imagecopy($im, $logo, 15, 15, 0, 0, $width, $height);
```

当将 logo 图片复制到原图片上以后，将加水印后的图片输出保存就完成了加水印处理。

```
    imagejpeg($im, $filename);
```


## 文件

### 上传／下载

#### PHP 简化后的上传原理

将客户端的文件上传到服务器端，然后将在服务器端的临时文件移动到指定文目录即可。

#### $_FILES

`$_FILES` 保存了上传文件的原始信息。

- **name：** 上传文件的名字
- **type：** 上传文件的 MIME 类型
- **tmp_name：** 上传到服务器后的临时文件名
- **size：** 上传文件的大小
- **error：** 上传文件的错误号


#### 浏览器客户端设置
 - 表单的 `enctype` 必须为：`multipart/form-data`。
 - 表单 `method` 必须为 post。
 
 如果 `method` 和 `enctype` 写错，则 `$_FILES` 为空。
 
#### php.ini 上传设置

##### File Uploads

- **file_uploads = On：** 支持 HTTP 上传。
- **upload_tmp_dir=：** 临时文件保存路径。
- **upload_max_filesize=2M：** 允许上传文件的最大值。
- **max_file_uploads=20，** 允许一次上传的最大文件数。
- **post_max_size = 8M：** POST 提交数据的最大值。

##### Resource Limits
 - max_execution_time = -1
 设置脚本被解析器终止之前允许的最大执行时间，单位为秒，防止烂代码写得不好而占用服务器资源。
 
 - max_input_time = 60
 脚本解析输入数据允许的最大时间，单位为秒。
 
 - max_input_nesting_level = 64
 设置输入变量的嵌套深度。
 
 - max_input_vars = 1000
 接受多少输入的变量，本限制将作用于 `$_GET`/`$_POST`/`$_COOKIE`。
 
 该指令可以减轻以哈希碰撞来进行 DDOS 攻击的可能性。如果有超过该指令限制数量的变量，将会导致 E_WARNING 错误，更多的输入变量将会从请求中截断。
 
 - memory_limit = 128M
 最大单线程的独立内存使用量。也就是一个 web 请求，给予线程最大的内存使用量的定义。

#### 错误信息说明

 - UPLOAD_ERR_OK/0：文件上传成功。
 - UPLOAD_ERR_INI_SIZE/1：上传文件超过 upload_max_filesize 选项的限制。
 - UPLOAD_ERR_FORM_SIZE/2：上传文件大小超过了 HTML 表单中 MAX_FILE_SIZE 选项的值。
 - UPLOAD_ERR_PARTIAL/3：部分文件被上传。
 - UPLOAD_ERR_NO_FILE/4：没有文件被上传。
 - UPLOAD_ERR_NO_TMP_DIR/6：找不到临时路径。
 - UPLOAD_ERR_CANT_WRITE/7：文件写入失败。
 - UPLOAD_ERR_EXTENSION/8：上传的文件被 PHP 扩展程序中断。
 
#### 移动上传成功的文件

通过 `move_uploaded_file()` ，该函数对安全模式和 open_basedir 都是敏感的。不过，限制只针对 destination 路径，因为允许移动上传的文件名 filename 可能会与这些限制产生冲 突。

`move_uploaded_file()` 仅作用于通过 PHP 上传的文件以确保这个操作的安全性。

**提醒：** 如果目标文件已经存在，将会被覆盖。

#### 上传文件限制

##### 客户端限制

```
    <!-- 通过表单隐藏域限制上传文件的最大值 -->
    <input type="hidden" name="MAX_FILE_SIZE" vlaue="字节数">
    
    <!-- 通过 accept 表单属性限制上传文件类型 -->
    <input type="file" name="upload_key" accept="文件的 MIME 类型">
    
    <!-- 通过 multiple 表单属性限制上传文件个数 -->
    <input type="file" name="upload_key" multiple="multiple">
    
    <!-- 通过 数组形式 表单名限制上传文件个数 -->
    <input type="file" name="upload_keys[]">
    
    <!-- 通过多个 file 控件限制上传文件个数 -->
    <!-- 实质也是单文件上传, 此时 $_FILES 是一个二维数组 -->
    <input type="file" name="upload_key1">
    <input type="file" name="upload_key2">
```

**提醒：** 客户端做的限制并不安全，因为可以在客户端任意更改代码。

##### 服务器限制

 - 限制上传文件的大小
 - 限制上传文件的类型
 - 检测是否问需要的文件类型
 - 检测是否为 HTTP POST 方式上传

### 文件系统

#### 读取文件内容

PHP 具有丰富的文件操作函数，最简单的读取文件的函数为 `file_get_contents`，可以将整个文件全部读取到一个字符串中。

```
    $content = file_get_contents('./test.txt');
```

`file_get_contents` 也可以通过参数控制读取内容的开始点以及长度。

```
    $content = file_get_contents('./test.txt', null, null, 100, 500);
```

PHP 也提供类似于 C 语言操作文件的方法，使用 fopen，fgets，fread等方 法，fgets 可以从文件指针中读取一行，freads 可以读取指定长度的字符串。

```
    // 1.fgets
    $fp = fopen('./text.txt', 'rb');
    while(!feof($fp)) {
    	echo fgets($fp); // 读取一行
    }
    fclose($fp);
    
    // 2.fread
    $fp = fopen('./text.txt', 'rb'); $contents = ''; while(!feof($fp)) {
    $contents .= fread($fp, 4096); //一次读取4096个字符 }
    fclose($fp);
```

使用 fopen 打开的文件，最好使用 fclose 关闭文件指针，以避免文件句柄被占用。


#### 判断文件是否存在

一般情况下在对文件进行操作的时候需要先判断文件是否存在，PHP 中常用来判断文件存在的函数有两个 `is_file` 与 `file_exists`。

```
    $filename = './test.txt';
    if (file_exists($filename)) {
      echo file_get_contents($filename);
    }
```

如果只是判断文件存在，使用 `file_exists` 就行，`file_exists` 不仅可以判断文件是否存在，同时也可以判断目录是否存在。
而从函数名可以看出，`is_file` 是确切的判断给定的路径是否是一个文件。

```
    $filename = './test.txt';
    if (is_file($filename)) {
      echo file_get_contents($filename);
    }
```

更加精确的可以使用 `is_readable` 与 `is_writeable` 在文件是否存在的基础上，判断文件是否可读与可写。

```
    $filename = './test.txt';
    if (is_writeable($filename)) {
      file_put_contents($filename, 'test');
    }
    if (is_readable($filename)) {
      echo file_get_contents($filename);
    }
```

#### 写入内容到文件

与读取文件对应，PHP 写文件也具有两种方式，最简单的方式是采用 `file_put_contents`。

```
    $filename = './test.txt';
    $data     = 'test';
    file_put_contents($filename, $data);
```

上例中，$data 参数可以是一个一维数组，当 $data 是数组的时候，会自动的将数组连接起来，相当于 `$data=implode('', $data)`;

同样的，PHP 也支持类似 C 语言风格的操作方式，采用 `fwrite` 进行文件写入。

```
    $fp = fopen('./test.txt', 'w');
    fwrite($fp, 'hello');
    fwrite($fp, 'world');
    fclose($fp);
```

#### 取得文件的修改时间

文件有很多元属性，包括：文件的所有者、创建时间、修改时间、最后的访问时间等。

 - **fileowner：** 获得文件的所有者
 - **filectime：** 获取文件的创建时间
 - **filemtime：** 获取文件的修改时间
 - **fileatime：** 获取文件的访问时间

其中最常用的是文件的修改时间，通过文件的修改时间，可以判断文件的时效性，经常用在静态文件或者缓存数据的更新。

```
    $mtime = filemtime($filename);
    echo '修改时间:'.date('Y-m-d H:i:s', filemtime($filename));
```

#### 取得文件的大小

通过 `filesize` 函数可以取得文件的大小，文件大小是以字节数表示的。

```
    $filename = '/data/webroot/usercode/code/resource/test.txt';
    $size = filesize($filename);
```

如果要转换文件大小的单位，可以自己定义函数来实现。

```
    function getsize($size, $format = 'kb') {
      $p = 0;
    if ($format == 'kb') {
      $p = 1;
    } elseif ($format == 'mb') {
      $p = 2;
    } elseif ($format == 'gb') {
      $p = 3;
    }
    $size /= pow(1024, $p);
    return number_format($size, 3);
    $filename = '/path/to/test.txt';
    $size = filesize($filename);
    $size = getsize($size, 'kb'); // 进行单位转换
    echo $size.'kb';
```

值得注意的是，没法通过简单的函数来取得目录的大小，目录的大小是该目录下所有子目录以及文件大小的总和，因此需要通过递归的方法来循环计算目录的大小。

#### 删除文件

跟 Unix 系统命令类似，PHP 使用 `unlink` 函数进行文件删除。

删除文件夹使用 `rmdir` 函数，文件夹必须为空，如果不为空或者没有权限，则会提示失败。

如果文件夹中存在文件，可以先循环删除目录中的所有文件，然后再删除该目录，循环删除可以使用 `glob()` 函数遍历所有文件

```
    unlink($filename);
    rmdir($dir);
    foreach (glob('*') as $filename) {
      unlink($filename);
    }
```

### 文件流 (IO)

从磁盘到内存，是 I；从内存到磁盘，是 O。


## OOP

### what ?

把生活中问题的解决方案，都封装成对象的方式：属性+行为/方法，进行存储。

### 为什么软件设计强调高内聚低耦合?

 - 高内聚：该有的都有
 - 低耦合：代码复用性高。
 - 总结：有利于多人开发大型项目。
 
### 3 大特性

#### 继承

`继承`是面向对象程序设计中常用的一个特性，比如，汽车是一个比较大的类，我们也可以称之为基类，除此之外，汽车还分为卡车、轿车、东风、宝马等，因为这些子类具有很多相同的属性和方法，可以采用继承汽车类来共享这些属性与方法，实现代码的复用。

##### 继承的好处
 - 父类中定义过的类成员不用再子类中重复定义，子类可以直接通过 $this-> 调用父类成员，节省了编程时间。
 - 同一个父类的子类拥有和父类相同的类成员，因此外部代码调用它们的时候可以一视同仁。
 - 子类可以修改和调整父类定义过的成员，即实现重写。最终执行的是子类重写后的成员。PHP 中只能单继承而不能像 `C++` / `Python` 可以多重继承。
 
#### 封装

体现封装性的地方有：

##### 访问控制

 - **public：** 可以被类自身、子类、其他类、以及类实例化后的对象访问。
 - **protected：** 可以被类自身、子类访问。
 - **private：** 只能被父类成员访问。但是可以被可以访问它的类中公共成员函数向外提供一个可以 访问的接口。
 
##### 接口

**类是对象的抽象，接口是类的抽象**，因此接口其实也是一种抽象类。

**接口不能够用于实例化对象**。一个类中如过实现了某个接口，那么该类必须实现接口中定义的方法，而接口中不能够具体实现定义的方法。举例说明：

```
    interface InterfaceName
    {
    	public function func($parameter);
    }
```

那么如果一个类被告知要实现该接口，就必须实现 func 方法：

```
    class A implements InterfaceName
    {
      public function func($parameter)
      {
    	// do something
      }
    }
```

###### 如何判断某个对象是否实现了某个接口？

```
    var_dump($obj instanceof InterfaceName);
```

###### 一个接口能继承另一个接口吗？

能。但是因此实现该接口的类必须实现子接口和父接口中的所有方法。

##### 抽象类

面向对象中的传统抽象类是抽象程度介于接口和类之间的一种表示形态。即抽象类中，部分方法可以实现，部分方法也可以不被实现而让子类去实现。

举例说明：

```
    abstract class AbstractClassName
    {
      abstract public function func1($parameter);
      public function func2()
      {
        // do something
      }
    }
    class ClassName extends AbstractClassName
    {
      // func1 必须被子类实现 func2 不必
      public function func1($parameter )
      {
    	// do something here
      }
    }
```

###### 抽象类存在的必要性

接口中某些方法可能部分对于实现它的类来说都是一样的，而部分又是体 现多态性的，因此对于实现该接口的所有类来说，就算所有类的某个方法本质都是一样的，也 必须实现该方法，因为该方法在接口中定义过了，这样如果实现该接口的类很多，重复的代码 量也就越大。

所以，如果把许多类都具备的方法都实现在接口中，那么将减少很多多余的编码工作，但为了不改变接口出现的概念和意义，因此在面向对象程序设计中，应时出现了抽象类 这一概念，完美地解决了这一问题。

抽象类中，abstract 修饰过的方法就必须被继承该抽象类的子类所实现，而其他方法直接在抽象类中实现。抽象类的定义也需要使用 abstract class 关键字修饰。

#### 多态

##### 多态的表现
 - 接口的方法可以被不同的类根据具体需求不同地实现。
 - 父类的方法可以被子类重写以实现不同的功能。
 - 同一个函数名可以根据参数的不同而表现不同的功能。
 
举例说明：
```
    interface I
    {
    	public function foo($v);
    }
    class A implements I
    {
      public function foo($v)
      {
          // do something a
      }
    }
    class B implements I
    {
      public function foo($v) {
    	  // do something b
      }
    }
    function func($obj)
    {
    	if( $obj instanceof I )
        {
    		$obj -> foo( 'v' );
        } else {
    		// do something
        }
    }
    $a = new A();
    $b = new B();
    func( $a );
    func( $b );
```

在上面这段代码中，函数 func() 通过传入的参数不同，执行不同的动作，这就体现了多态。

##### 抽象程度

根据上面的讨论可得：`对象` -> `类` -> `抽象类` -> `接口`，抽象程度逐渐上升。


### 类的属性

在类中定义的变量称之为属性，属性声明是由关键字 `public`，`protected` 或者 `private` 开头，后面跟一个普通的变量声明来组成。属性的变量可以设置初始化的默认值，默认值必须是常量。

#### 如何访问成员？

成员分为**属性**和**方法**。

类属性类型默认都为 public，外部可以访问。一般通过 `->` 对象操作符来访问对象的属性或者方法。对于静态属性则使用 `::` 双冒号进行访问。当在类成员方法内部调用的时候，可以使用 `$this` 伪变量调用当前对象的属性。

`$this` 是 PHP 等语言中的伪变量，表示对象自身。类中可以通过 `$this->member`，`$this->fun()`来调用类成员。但是 `member` 和 `fun()`前面不能有 `$`。

### 类方法

方法就是在类中的 function，很多时候我们分不清方法与函数有什么差别，
 - 在面向过程的程序设计中function 叫做**函数**，
 - 在面向对象中 function 则被称之为**方法**。

同属性一样，类的方法也具有 public，protected 以及 private 的访问控制。使用关键字 static 修饰的，称之为`静态方法`，静态方法不需要实例化对象，可以通过类名直接调用，操作符为双冒号 `::`。

```
    class Car
    {
      public static function getName()
      {
        return 'Car';
      }
    }
    echo Car::getName();
```

#### 构造函数

PHP5 可以在类中使用 `__construct()` 定义一个构造函数，具有构造函数的类，会在每次对象创 建的时候调用该函数，因此常用来在对象创建的时候进行一些初始化工作。在子类中如果定义了 `__construct()` 则不会调用父类的 `__construct()`，如果需要同时调用父类的构造函数，需要使用 `parent::__construct()` 显式的调用。

```
    class Car
    {
    	public function __construct()
        {
          print "父类构造函数被调用\n";
        }
    }
    class Truck extends Car
    {
    	publich function __construct()
        {
    		print "子类构造函数被调用\n";
          	parent::__construct();
    	}
    }
    $car = new Truck();
```

#### 析构函数

同样，PHP5 支持析构函数，使用 `__destruct()` 进行定义，析构函数指的是当某个对象的所有引用被删除，或者对象被显式的销毁时会执行的函数。**析构函数的作用：用于释放某些资源。**

```
    class Car
    {
    	public function __construct()
        {
    		print "构造函数被调用 \n";
        }
    	public function __destruct()
        {
    		print "析构函数被调用 \n";
    	}
    }
    $car = new Car(); // 实例化时会调用构造函数
    echo '使用后，准备销毁car对象 \n';
    unset($car);     // 销毁时会调用析构函数
```

同构造函数，如果不认为指定，将执行默认的析构函数。如果人为指定了，则覆盖默认方法。

*但是析构函数的触发条件有两个：*
 - 程序执行结束，系统自动调用析构函数
 - 当对象不再被使用时，比如赋值为 null 时。但更精确的说法是：当对象不再被某个变量所引用时，就会自动调用析构函数。
 
> 或者说，对象没有任何变量引用时和程序运行结束时，就会触发析构函数。

当 PHP 代码执行完毕以后，会自动回收与销毁对象，因此一般情况下不需要显式的去销毁对象。

### 对象引用

引用和指针作用很类似。都是指向某个地址，比如指向对象的话，那么就指向堆中的某个地址。

#### &

如果不用引用，那么如果把对象赋值给另一个变量时，对原变量的改变不会影响另一个变量，所以也不会影响到这个对象，即两个变量代表两个不同的地址，即具有拷贝属性。

如果使用对象引用，同样的情况发生时，无论是对哪个变量的改变都是对同一个对象的改变。即两个变量指向的都是同一个对象在内存中的地址。

举例说明：

 - 不使用引用
 
 ```
    $a = new C();
    $b = $a;
    $a = null;
 ```
 
 此时对象的析构函数不会自动调用，因为还有 `$b` 指向该对象。
 
 - 使用引用
 
 ```
    $a = new C();
    $b = &$a;
    $a = null;
 ```
此时对象的析构函数便会自动调用。

综上，如果 `$a` = `$b`，那么从内存上来看，`$a` 和 `$b` 是两块不同的内存地址空间，即 `$a` 和 `$b` 是不同的两个引用。而如果 `$a` = `&$b`，那么 `$a` 和 `&$b` 代表的都是同一块内存空间，即 `$a` 和 `$b` 是同一个引用。


### 数据访问修饰符

访问控制通过关键字 `public，protected 和 private` 来实现。

 - 被定义为公有的类成员可以在任何地方被访问。
 - 被定义为受保护的类成员则可以被其自身以及其子类和父类访问。
 - 被定义为私有的 类成员则只能被其定义所在的类访问。
 
类属性必须定义为公有、受保护、私有之一。为兼容 PHP5 以前的版本，如果采用 var 定义，则被视为公有

类中的方法可以被定义为公有、私有或受保护。如果没有设置这些关键字，则该方法默认为公有。

```
    class Car
    {
      $speed = 10; // 错误 属性必须定义访问控制
      // 正确（但是不规范）
      function getSpeed()
      {
        return $this->$speed;
      }
    }
```

如果构造函数定义成了私有方法，则不允许直接实例化对象了，这时候一般通过静态方法进行实例化，在设计模式中会经常使用这样的方法来控制对象的创建，比如单例模式只允许有一个全局唯一的对象。

```
    class Car
    {
    	private function __construct()
        {
    		echo 'object create';
        }
    	private static $_object = null;
    	public static function getInstance()
        {
    		if (empty(self::$_object)) {
    			self::$_object = new Car(); // 内部方法可以调用私有方法，因此这里可以创建对象
    		}
    		return self::$_object;
        }
    }
    // $car = new Car(); // 错误 这里不允许直接实例化对象
    $car = Car::getInstance(); // 正确 通过静态方法来获得一个实例
```

### 如何在子类中访问被子类重写过的方法？

如果被 overwrite 的方法名为 func，则可以使用 `parent::func()`; 的形式来**显示地调用被重写的父类方法**。

### 如何在类中访问 const 修饰过的成员？

```
    const CONST_V = 1024;
    self::CONST_V;
```

**注意：** const 修饰的变量名前不需要写 $。


### `$this` 的使用场合

`$this->` 可用于在类中访问非 `static` 修饰的类成员。

`$this->member` 中的 `member` 前面不需要 `$` 而 `self::` 和 `static::` 需要 。

### static 关键字

静态属性与方法可以在不实例化类的情况下调用，直接使用 `类名::方法名` 的方式进行调用。静态属性不允许对象使用 `->` 操作符调用。

#### static 关键字在 OOP 中有什么用？

为了共用类中一些共有的相同成员属性或者方法，同时保存类的共有数据。

#### 如何在类中定义 static 成员？

static 放在访问控制符之后，即应该是如下形式：
```
    private static $member;
    public static function foo()
    {
      // ...
    }
```

#### 如何访问类中 static 修饰的成员？

类中被 static 修饰的成员不受类实例化的限制，即无论在哪里都可以直接访问。

- 类中访问自身 static 成员
类中访问自身 static 成员属性是用 `self::$member` 或者 `static::$member` (注意 $ )。

类中访问自身 static 成员方法是用 `self::func()` 或者 `static::func()`。

- 类外访问 static 成员

静态成员不需要实例化对象就可以访问。

类外访问 static 成员属性是用 `ClassName::$member_name` (注意 $ )。

类外访问 static 成员方法是用 `ClassName::func_name()`。

- 子类中访问父类中的 static 成员

子类中访问父类的 static 成员属性是用 `parent::$member_name`。(注意 $ )

子类中访问父类的 static 成员方法是用 `parent::func_name()`

#### 动态调用静态方法

静态方法也可以通过变量来进行动态调用。

```
    $func = 'getSpeed';
    $className = 'Car';
    echo $className::$func(); // 动态调用静态方法
```

#### 静态方法中可以访问非静态属性吗？

不能。静态方法里面只能访问静态属性。

#### `$this` 可以访问静态成员吗？

不能。静态方法中，`$this` 伪变量不允许使用。可以使用 `self`，`parent`，`static`在内部调用静态方法与属性。

```
    class Car
    {
    	private static $speed = 10;
    	public static function getSpeed()
        {
          return self::$speed;
    	}
    	public static function speedUp()
        {
          return self::$speed+=10;
    	}
    }
    class BigCar extends Car
    {
      public static function start()
      {
    	parent::speedUp();
      }
    }
```

由于 `$this` 的使用必须是在对象上下文环境中。所以不能出现这种用法：

```
    public static function func()
    {
      $this->member;
    }
```

### final 关键字

#### 为什么需要 final 关键字?

为了避免子类重写父类方法或者为了避免类被继承。

使用 final 举例：

```
    // 类 ClassName 不能被继承
    final class ClassName
    {
      // do something
    }
    
    // 方法 fun 不能被重写
    final public function fun()
    {
      // do something
    }
```

### 魔术方法

魔术方法是 PHP 特有的 ( 其他语言也有类似的实现 ) ，在某些情境下会触发而被自动调用的方法。

> “魔术” 二字可以大概的有种不可能的事情欺骗了我们的双眼而成为现实的感觉。
>  
>  总之，魔术方法可以将看似不能完成的事情实现的作用。

#### construct 和 destruct()

`对象创建时` 和 `对象销毁/程序结束时` 自动调用。

#### __toString()

当对象被当作 String 使用时自动调用。比如：`echo (new A())`;。

#### __invoke()

当对象被当作方法调用时自动调用。比如：`$obj($parameter)`;

#### __call() 和 __callStatic()

当对象调用类中不存在的方法或静态方法时自动调用。

这两个方法在 PHP 中也被叫做`方法的重载 ( overload )`。它们的定义类似：

```
    ???
```

其中 $name 代表的是被调用的那个不存在的方法名；$argvs 是一个数组，保存了调用该不存在的方法的时候传入的形参列表。

##### __call() 和 __callStatic() 有什么用？

可以实现已一种 PHP 传统语法无法定义的方式去定义一个具有多态性的方法。

因为同一个函数名的调用却可以因为不同的调用方式完成不同的功能。举例说明：

```
    class A
    {
      public function __call($name, $argvs)
      {
        // do something
      }
      public static function __callStatic($name, $argvs)
      {
        // do something
      }
    }
    $a = new A();
    $a -> test();
    $a :: test();
```

可以看出，通过 `__call()` 和 `__callStatic()`，就像调用了已经存在于类中的方法一样。

同时，又实现了以普通 PHP 语法无法实现的：同时定义一个同名的 static 方法和非 static 方法，完成不同的功能，这也体现了多态。

#### `__get()` / `__set()` / `__isset()` / `__unset()`

这 4 个方法在 PHP 中被称为属性重载的魔术方法。

 - **__get()：** 读取不可访问属性时自动调用。
 - **__set()：** 在给不可访问属性赋值时自动调用。
 - **__isset() : ** 在对不可访问属性调用 isset() 或者 empty() 时自动调用。可以通过自定义 __isset() 方法决定不同的返回值。
 - **__unset() :** 在对不可访问属性调用 __unset() 时自动调用。
 
##### `__get()` 和 `__set()` 有什么用？

可以动态地给类中添加/访问原本不存在的属性。

比如可以将该不可访问属性通过一定的方法保存在类中，如预定义的空数组。

以上 4 个方法可以灵活地操作类中原本不存在的数据以实现某些需求。

#### 什么是不可访问属性？

属性不存在或者不具备访问权限。

##### `__clone()`

对象复制时候自动调用。作用是屏蔽不想被复制的属性或者对对象副本进行属性的初始化等操作。

**举例说明：**

```
    class A
    {
      public $property;
      public function __clone()
      {
        // do something
        $this->property = 'temp';
      }
    }
    $a0 = new A();
    $a0 -> property = 'value0';
    $a1 = clone $a0 ;
    echo"Beforesetting: $a1->property= ", $a1->property;
    $a1 -> property = 'value1';
    echo"Aftersetting: $a1->property= ", $a1->property;
    
```

**说明：** `clone` 的出现是为了解决通过引用或者变量赋值的方式不能真正复制对象的不足。指向对象的引用或 者变量代表的仅仅是对象在内存中的存放位置。


### 重载

PHP 中的重载指的是动态的创建属性与方法，是通过魔术方法来实现的。

#### 属性的重载

属性的重载通过 `__set()`，`__get()`，`__isset()`，`__unset()` 来分别实现对不存在属性的赋值、读取、判断属性是否设置、销毁属性。

```
    class Car
    {
      private $ary = array();
      public function __set($key, $val)
      {
        $this->ary[$key] = $val;
      }
      public function __get($key) {
        if (isset($this->ary[$key])) {
          return $this->ary[$key];
        }
      	return null;
      }
      public function __isset($key) {
        if (isset($this->ary[$key])) {
          return true;
        }
      	return false;
      }
      public function __unset($key) {
        unset($this->ary[$key]);
      }
    }
    $car = new Car();
    $car->name = '汽车'; // name属性动态创建并赋值
    echo $car->name;
```

#### 方法的重载

方法的重载通过 `__call()` 来实现，当调用不存在的方法的时候，将会转为参数调用 `__call()` 方法，当调用不存在的静态方法时会使用 `__callStatic()` 重载。


```
    class Car {
      public $speed = 0;
      public function __call($name, $args)
      {
        if ($name == 'speedUp') {
          $this->speed += 10;
        }
      }
    }
    $car = new Car();
    $car->speedUp(); // 调用不存在的方法会使用重载
    echo $car->speed;
```

#### 如何理解重载？

可以通过“重定向”这一概念来理解。

重定向指的是原本要请求某个 URL 时，由于源资源不存在或者其他原因，服务器返回的却是另一个 URL 的资源。

- **方法重载**指的是原本要访问某个成员方法的时候，由于方法不存在或其他原因，PHP 解析引擎实际调用的却是另一个方法。
- **属性重载**指的是原本要访问的某个成员属性，由于不存在或者其他原因，实际是通过相应的魔术方法完成对类中不存在的成员属性进行的相关操作，就相当于类中有该属性，并相当于在操作类中存在的属性。

### 对象的高级特性

#### 对象比较

当同一个类的两个实例的所有属性都相等时，可以使用比较运算符 `==` 进行判断，当需要判断两个变量是否为同一个对象的引用时，可以使用全等运算符 `===` 进行判断。

```
    class Car {}
    $a = new Car();
    $b = new Car();
    if ($a == $b) echo '=='; // true
    if ($a === $b) echo '==='; // false
```

#### 对象复制

在一些特殊情况下，可以通过关键字 clone 来复制一个对象，这时 __clone 方法会被调用，通过这个魔术方法来设置属性的值。

```
    class Car
    {
      public $name = 'car';
      public function __clone()
      {
        $obj = new Car();
        $obj->name = $this->name;
      }
    }
    $a = new Car();
    $a->name = 'new car';
    $b = clone $a;
    var_dump($b);
```

### 对象序列化

可以通过 `serialize` 方法将对象序列化为字符串，用于存储或者传递数据，然后在需要的时候通过 `unserialize` 将字符串反序列化成对象进行使用。

```
    class Car
    {
    	public $name = 'car';
    }
    $a = new Car();
    $str = serialize($a); // 对象序列化成字符串
    echo $str, PHP_EOL;
    $b = unserialize($str); // 反序列化为对象
    print_r($b);
```

## 数据库

### PHP 支持哪些数据库

PHP 通过安装相应的扩展来实现数据库操作，现代应用程序的设计离不开数据库的应用，当前主流的数据库 有 MsSQL，MySQL，Sybase，DB2，Oracle，PostgreSQL，Access 等，这些数据库 PHP 都能够安装扩展来支持。

一般情况下常说的 LNMPA 架构指的是：Linux、Nginx、MySQL、PHP、Apache，因此 MySQL 数据库在 PHP 中的应用非常广泛。

### 数据库扩展

PHP 中一个数据库可能有一个或者多个扩展，其中既有官方的，也有第三方提供的。

像 MySQL 常用的扩展有原生的 mysql 库（新版本已弃用），也可以使用增强版 的 mysqli 扩展，还可以使用 PDO 进行连接与操作。不同的扩展提供基本相近的操作方法，不同的是可能具备一些新特性，以及操作性能可能会有所不同。

举例，判断 mysql 数据库扩展是否已经安装：

```
    if (function_exists('mysql_connect')) {
    	echo 'mysql 扩展已经安装';
    }
```

### PDO

> PDO （PHP Data Object）是个当前 PHP 操作数据库的主要操作扩展。也建议必须掌握这个的使用。

#### why ?

为了完成相同的功能，PHP 在操作不同的数据库的时候需要使用不同的 API，维护和升级不同数据库的时候变得更繁琐。

#### what ?

PDO 是数据库访问抽象层，统一各种数据库的访问接口。具有如下特性：

 - 编码一致性
 PDO 提供操作各种数据库的统一接口。
 
 - 灵活性
 PDO 在运行时会加载数据库驱动程序，而不需要在每次使用不同的数据库时再去配置和编译 PHP。
 
 - 高性能
 PDO 使用 C 语言写的，编译为 PHP。
 
 - OOP 特性
 PDO 是一个抽象接口，利用 PDO 本身并不能实现对数据库的操作。还需要具体的实现方式
 
 > PDO 支持的所有数据库见：http://php.net/manual/zh/pdo.drivers.php。
 
##### DSN

Data Source Name：`驱动`:`主机名`;`数据库`。

#### how ?

##### 启用和配置 PDO 扩展

```
    1. 配置 php.ini 开启相应的扩展
    extension = php_pdo.dll
    2. 开启相应的数据库扩展，如 MySQL
    extension = php_pdo_mysql.dll
    3. 重启 php-fpm(nginx) 或者 apahce
```

其余操作详见手册：[PHP 数据对象](http://php.net/manual/zh/book.pdo.php#book.pdo)。


#### 和 mysqli/mysql 对比

PDO 连接 MySQL 的效率和插入数据比 mysql/mysqli 低，但是插入数据的效率没有连接数据库差别那么大。

### mysql && mysqli

#### mysqli 扩展与 mysql 扩展的区别

mysql 和 mysqli 都是 PHP 操作 MySQL 数据库的扩展，mysqli 是mysql 的扩展版本，两者 API 命名上的差别基本上只是一个 i，因此，掌握 mysqli 就掌握了 mysql。

对于 4.0.1 版本后的 MySQL，mysql 扩展对其新特性的支持不好，PHP5 之后官方推荐使用 mysqli 和 PDO 方式来操作 MySQL。

#### mysqli 相对于 mysql 扩展的优点：
 
 - 支持两种编程风格：面向对象和面对过程
 - 支持预处理语句
 - 支持事务
 - 速度比 MySQL 扩展快
 - 安全性提高
 - 支持 MySQL 的新特性
 
#### 配置 mysqli

 - 开启 php_mysqli.dll 扩展：php.ini 中取消 php_mysqli.dll 前面的注释。
 - 配置 `extension_dir=`：指定到 ext 目录的实际路径。
 - 重启服务器后测试
 
 ```
    phpinfo();
    var_dump(extension_loaded('mysqli'));
    var_dump(function_exists('mysqli_connect'));
    print_r(get_loaded_extensions());
 ```

#### mysqli 基本使用

PHP 对扩展的使用基本上都有 2 种方式：面向对象和面向过程，mysqli 也不例外，详见手册。

#### 关于 SQL 操作方法的返回值

 - select / desc / describe / show / explain 等执行成功返回 mysqli_result 对象，失败返回 false。
 - 其他 SQL 语句执行成功返回 true，执行失败返回 false。
 
#### 基本步骤

##### 连接、选择数据库

```
    $mysqli = new mysqli( $host, $user, $passwd, $db ) ;
    // 也可以通过 mysql_select_db 在连接成功手动选择数据库
    mysqli_select_db('db_name');
    if( $mysqli->connect_errno ) {
    	die( $mysqli->connect_error ) ;
    }
```

##### 设置客户端默认字符集／编码方式

```
    $myslqi->set_charset('utf8');    // not `utf-8`
```

##### 执行 SQL 语句，根据情况对执行结果进行处理
    
```
    $res = $mysqli->query( $sql ) ;
    // 这里的重点还是 SQL 语句和对结果集的处理
    if( $res && $res->num_rows>0 ) {
        // $res->fetch_assoc() ; // 默认抓取结果集中的一个由关联数组和索引数组组成的二维数组
        // $res-> fetch_all( MYSQL_ASSOC ) ;
        while( $row = $res->fetch_assoc() ) {
            $rows = $row ;
        }
    }
```

默认的，PHP 使用最近的数据库连接执行查询，但如果存在多个连接的情况，则可以通过参数指令从那个连接中进行查询。

```
    $link1 = mysql_connect('127.0.0.1', 'code1', '');
    $link2 = mysql_connect('127.0.0.1', 'code1', '', true); // 开启一个新的连
    接
    $res   = mysql_query('select * from user limit 1', $link1); // 从第一个连接中查询数据
```

##### 关闭连接

当数据库操作完成以后，可以使用 mysql_close 关闭数据库连接。

默认的，当 PHP 执行完毕以后，会自动的关闭数据库连接，一般情况下已经满足需求，但是在对性能要求比较高的情况下，可以在进行完数据库操作之后尽快关闭数据库连接，以节省资源，提高性能。

```
    mysqli_close();
```

在存在多个数据库连接的情况下，可以设定连接资源参数来关闭指定的数据库连接。

```
    $link = mysql_connect($host, $user, $pass);
    mysql_close($link);
```

#### mysqli 多语句查询

##### 插入多条语句

需要把需要执行的 SQL 语句用分号分开。

```
    $sql  = "select ... ;" ;
    $sql .= "update ...;" ;
    $sql .= "delete from ..." ;
    $mysqli -> multi_query( $sql ) ;
```

第一条 SQL 语句执行的成功与否影响了 mutlti_query() 的返回值，第一条执行成功，后面的不论执行失败与否该方法都返回 true。

每条 SQL 语句如果执行成功都会返回一个结果集。

 - **use_result() / store_result()：** 获取第一条查询产生的结果集
 - **more_results()：** 检测是否有更多的结果集
 - **next_result()：** 将结果集指针移动到下一个结果集。
 
**举例说明：**
    
```
    if ($mysqli->multi_query($sql)) {
        do {
            if ($mysqli_result = $mysqli->store_result()) {
                $rows[] = $mysqli_result->fetch_assoc() ;
            }
        } while ($mysqli->more_results() && $mysqli->next_result()) ;
    } else {
        $myslqli->error ;
    }
```

#### mysqli 方法／函数

```
    $mysqli->connect_error
    $mysqli->connect_errno
    $mysqli->client_info
    $mysqli->client_version
    $mysqli->get_server_info()
    $mysqli->insert_id()    // 得到上一次操作产生的 AUTO_INCREMENT 的值
    $mysqli->affected_rows()    // 得到上一次操作产生的受影响的条数
```

##### affected_rows() 的返回值说明
 
 - -1 代表 SQL 语句有问题、
 - 0 代表没有受影响记录的条数
 - 其余情况下为受影响的记录条数
 
##### 结果集处理
    
```
    $res->fetch_all( MYSQLI_NUM/MYSQLI_ASSOC/MYSQLI_BOTH )$res->fetch_row() // 返回的是索引数组，取得的是结果集中的第一条
    记录作为索引数组返回
    $res->fetch_assoc() // 返回的是关联数组，取得的是结果集中的第一条记录作为关联数组返回，每 fetch/关联 一次，结果集中的指针就下移一位，当指针指向的最后一条后返回 false
    $res->fetch_array() // 上面二者都有，也可以和 fetch_all 一样限定返回类型
    $res->fetch_object() ; // 返回对象
    $res->data_seek( 2 ) ; // 移动结果集内部指针
    $res->close() ;    // 释放结果集
    $res->free();
    $res->free_result() ;
```

#### mysqli 预处理语句

mysqli 使用预处理语句需要通过在 SQL 语句借助 `占位符?` 实现。

**举例说明:**

```
    $sql = "insert user( user, passwd, age ) values( ?, ?, ? )" ;
    
    # 准备预处理语句
    $mysqli_stmt = $mysqli->prepare( $sql ) ;
    $user = '' ;$passed = '' ;$age = '' ;
    
    # 绑定参数:s 代表字符串;i 代表整型;d 代表浮点数$mysqli_stmt = bind_param( 'ssi', $user, $passed, $age ) ;
    
    # 执行预处理语句
    if( $mysqli_stmt -> execute() ) {echo $mysqli_stmt->insert_id ;echo '<br>' ;
    } else {
    	echo $mysqli_stmt -> error ;
    }
    
    # 释放结果集
    $mysqli_stmt -> free_result() ;
    
    # 关闭预处理语句
    $mysqli_stmt -> close ();
    
    # 关闭连接
    $mysqli -> close() ;
```

##### 使用预处理语句防止 SQL 注入

 - **不用预处理的 SQL 注入现象**
 
 ```
    $user = $_POST[ 'user' ] ;
    $passwd = md5( $_POST[ 'passwd' ] ) ;
    $sql = "select * from user where username='{$user}' and
    passwd='{$passwd}'" ;
    $res = $mysqli->query( $sql ) ;
    if( $res && $res->num_rows > 0 ) {
    	echo '登录成功' ;
    } else {
    	echo '登录失败' ;
    }
 ```
 
 此时如果在页面传递的是 `' or 1=1 # `那么 SQL 注入是成功的。
 
 - **使用预处理后防止 SQL 注入成功**
 
 ```
    $sql = "select * from user where username=? and passwd=?" ;
    $stmt = $mysqli->prepare( $sql ) ;
    if( $stmt->execute() ) {
      $stmt -> store_result() ;
      if( $stmt->num_rows>0 ) {
          echo '登录成功' ;
      } else {
          echo '登录失败' ;
      }
    }
 ```
 
##### 使用预处理语句执行查询操作

```
    $sql = "select id, user, age from user where id>=?" ;
    
    $id = 20 ;
    
    $mysql_stmt -> bind_param( 'i', $id ) ;
    
    if( $mysqli_stmt -> execute() ) {
      # 绑定结果集中的值到变量
      $mysqli_stmt->bind_result( $id, $user, $age ) ;
      while( $mysqli_stmt -> fetch() ) {
      	// do something
      }
    }
    
    $mysqli_stmt -> free_result() ;
    $mysqli_stmt -> close() ;
    $mysqli->close() ;
```

#### mysqli 操作事务

 - 数据准备
 
 ```
    create table account(
      id tinyint unsigned auto_increment primary key,
      username varchar(20) not null unique,
      money float( 6,2 )
    );
    insert account(user, money)
    values('test', 100), ('tset', 100);
 ```
 
 - PHP 代码
 
 ```
    # 关闭自动提交功能
    $mysqli->autocommit( FALSE ) ;
    
    $sql0 = "update account set money=money-200 where user='test'" ;
    $res0 = $mysqli->query( $sql0 ) ;
    $res0affect = $mysqli->affected_rows ;
    
    $sql1 = "update account set money=money-200 where user='tset'" ;
    $res1 = $mysqli->query( $sql1 ) ;
    $res1affect = $mysqli->affected_rows ;
    
    if( $res0 && $res0affect>0 && res1 && res1affect>0) {
      $mysqli->commit();
      echo '转账成功' ;
      # 恢复自动提交功能
      $mysqli->autocommit( true );
    } else {
      $mysqli->rollback() ;
      echo '转账失败' ;
    }
    
    $mysqli->close();
 ```
 
#### 其他

##### myslqi_select_db() 与 mysqli::select_db()

面向对象风格和面向过程风格在对函数的使用方式上是不同的。

如果采用面向对象的使用方式，那么要使用某个对象的方法，如果该方法不是静态方法， 则都需要先实例化一个 mysqli 对象，然后通过该对象调用类中的方法。

所以如果直接使用 `mysqli:select_db()` 就会出现不存在该静态方法的错误提示。

如果采用面向过程的使用方式，按照手册，则必须要有 2 个参数：第一个参数是 MySQL 链 接的资源描述符。即应该这么使用：

```
    $link = mysqli( $host, $user, $pwd ) ;
    mysqli_select_db( $link, $db_name ) ;
```

##### 基本函数的使用和解释

##### 4 大 fetch 函数解释

 - mysqli_fetch_row()
 
 每执行一次 mysqli_fetch_row() 函数，就从返回的结果集中取出一条( 行 - row )数据，并保存在一个一维索引数组中。
 
 可以使用循环打印出结果集中的查询到的所有数 据:
 
 ```
    $res = mysqli_query( $link, $sql ) ;
    while( $row = mysqli_fetch_row( $res ) ) {
      print_r( $row ) ;
    }
 ```
 
 > 这里刚开始可能会有点迷惑:为什么没有类似 i++ 的参数控制循环自己却正确地运行呢? 可以理解为 mysqli_fetch_xxx() 函数自带了这种判断，即没执行一次该函数后， 下一次执行该函数时，自动去取下一条数据，就像自增长的指针一样。如果返回值为 空，则说明查询语句的结果数据已经全部取完。
 
 
 - mysqli_fetch_array()
 
 mysqli_fetch_array() 与 mysqli_fetch_row() 有相同之处，但是不同之处在于：
 
 - **mysqli_fetch_row()** 取一条数据产生的是一个索引数组，保存的是数据库中字段 对应的值。
 
 - **mysqli_fetch_array()** 取一条数据默认返回的结果是一个索引数组和一个关联数组，其中索引数组中保持的也是数据库表中字段对应的值，关联数组中保存的就是数据库中字段的键名。有了关联数组，想要查询结果集中的某个数据，就可以 直接通过 $arr[ ‘key’ ] 的形式来直接获取了。
 
 不难看出，mysqli_fetch_array() 的执行速度比 mysqli_fetch_row() 慢很多，但是可以在执行 mysqli_fetch_array() 时带一些额外的参数让其执行更具体的操作，从而提高效率：
 
 ```
    `MYSQL_ASSOC`：返回关联数组
    `MYSQL_BOTH`：和默认状态的结果是一样的
    `MYSQL_NUM`：返回数字数组(索引数组)
 ```
 
 - `mysql_fetch_assoc()`
 
 从结果集中获得一个关联数组。和 `mysqli_fetch_array( $res, MYSQL_ASSOC )` 的执 行结果是完全一样的。
 
 - `mysql_fetch_object()`
 
 从结果集中获得一个对象。因此要按照对象的规矩来取数据。
 
 **举例说明：**
 
 ```
    $res = mysqli_fetch_object( $sql ) ;
    
    print_r( $res ) ;
    
    echo $res -> key;
 ```
 
##### 其他函数

 - `mysqli_num_rows()`
 
 返回结果集中的个/行数;可以用于当结果集为空的时候避 免下一步操作造成的资源浪费。
 
 - `mysqli_result( $res, int $row_no, [int $offset | string $field_name] )`
 
 返 回结果集的值，可以直接 echo。$res 是结果集的地址，$row_no 是行号，第三个参 数要么是偏移量 $offset 要么是字段名$field_name。
 
 Start 和 offset 的下标都从 0 开始。
 
 - `mysqli_affected_rows( $link )`
 
 返回前一句增删改 SQL 中受到影响的行数。这 里的参数是链接描述符而不是结果集地址。
 
##### 索引数组和关联数组

区别在于对数组内元素的访问方式不一样：
 
  - 索引数组：`$arr[ index ]`，其中 index 是整数。
  
  - 关联数组：`$arr[ "key" ]`，其中 key 是关联数组中键值对中的键名。
  
 
### 分页查询

通过循环可以获取一个查询的所有数据，在实际应用中，我们并不希望一次性获取数据表中的所有数据，那样性能会非常的低，因此会使用翻页功能，比如每页仅显示 10 条或者 20 条数据。

通过 mysql 的 limit 可以很容易的实现分页，`limit m,n` 表示从 m 行后取 n 行数据，在 PHP 中我们需要构造 m 与 n 来实现获取某一页的所有数据。假定当前页为 $page，每页显示 $n 条数据，那么 m 为当前页前面所有的 数据，即 `$m = ($page-1) * $n` ，在知道了翻页原理以后，那么我们很容易通过构造 SQL 语句在 PHP 中实现数据翻页。

```
    $page = 2;
    $n = 2;
    $m = ($page - 1) * $n;
    $sql = "select * from user limit $m, $n";
    $result = mysqli_query($sql);
    $data = array();
    // 循环获取当前页的数据
    while ($row = mysqli_fetch_assoc($result)) {
    	$data[] = $row;
    }
```

这里使用了 `$m` 与 `$n` 变量来表示偏移量与每页数据条数，但我们推荐使用更有意义的变量名来表示，比如`$pagesize`, `$start`, `$offset` 等，这样更容易理解，有助于团队协作开发。
 
 
 
## 正则表达式

正则表达式是对字符串进行操作的一种逻辑公式，就是用一些特定的字符组合成一个规则字符串，称之为正则匹配模式。

```
    $p = '/apple/';
    $str = "apple banna";
    if (preg_match($p, $str)) {
    	echo 'matched';
    }
```

其中，`/apple/` 就是一个正则表达式，他用来匹配源字符串是否存在 *apple* 字符串。

### PCRE

PHP 中使用 PCRE 库函数进行正则匹配，比如上例中的 preg_match 用于 执行一个正则匹配，常用来判断一类字符模式是否存在。

使用正则表达式的目的是为了实现比字符串处理函数更加灵活的处理方式，因此跟字符串处理函数一样，其主要用来判断子字符串是否存在、字符串替换、分割字符串、获取模式子串等。

PHP 使用 PCRE 库函数来进行正则处理，通过设定好模式，然后调用相关 的处理函数来取得匹配结果。preg_match 用来执行一个匹配，可以简单的用来判断模式是否匹配成 功，或者取得一个匹配结果，他的返回值是匹配成功的次数 0 或者 1，在匹配到 1 次以后就会停止搜索。


### 正则表达式的基本语法

PCRE 库函数中，正则匹配模式使用分隔符与元字符组成，分隔符可以是非数字、非反斜线、非空格的任意字符。经常使用的分隔符是正斜线 (/)、hash符号(#) 以及取反符号(~)，例如:

```
    /foo bar/
    #^[^0-9]$#
    ~php~
```

如果模式中包含分隔符，则分隔符需要使用反斜杠(\)进行转义：`http:\/\/`。

如果模式中包含较多的分割字符，建议更换其他的字符作为分隔符，或用 `preg_quote` 进行转义。

```
    $p = 'http://';
    $p = '/'.preg_quote($p, '/').'/';
    echo $p;
```

分隔符后面可以使用模式修饰符，模式修饰符包括：i, m, s, x 等，例如使用 i 修饰符可以忽略大小写匹配。

```
    $str = 'Http://www.imooc.com/';
    if (preg_match('/http/i', $str)) {
    	echo '匹配成功';
    }
```

#### 元字符与转义

正则表达式中具有特殊含义的字符称之为元字符，常用的元字符有：

```
    \ 一般用于转义字符
    ^ 断言目标的开始位置(或在多行模式下是行首)
    $ 断言目标的结束位置(或在多行模式下是行尾)
    . 匹配除换行符外的任何字符(默认)
    [ 开始字符类定义
    ] 结束字符类定义
    | 开始一个可选分支
    ( 子组的开始标记
    ) 子组的结束标记
    ? 作为量词，表示 0 次或 1 次匹配。位于量词后面用于改变量词的贪婪 特性。 (查阅量词)
    * 量词，0 次或多次匹配
    + 量词，1 次或多次匹配
    { 自定义量词开始标记
    } 自定义量词结束标记
```

下面的 \s 匹配任意的空白符，包括空格，制表符，换行符。

`[^\s]`代表非空白符。`[^\s]`+ 表示一次或多次匹配非空白符。

```
    $p = '/^我\s+(苹果|香蕉)$/';
    $str = '我喜欢吃苹果';
    if (preg_match($p, $str)) {
    	echo '匹配成功';
    }
```

元字符具有两种使用场景，`一种是可以在任何地方都能使用`，`另一种是只能在方括号内使用`。

在方括号内使用的有：
 
  - `\`：转义字符
  - `^`：仅在作为第一个字符(方括号内)时，表明字符类取反
  - `-`：标记字符范围
 
 其中 `^` 在反括号外面，表示断言目标的开始位置，但在方括号内部则代表字符类取反，方括号内的减号 `-` 可以标记字符范围，例如 0-9 表示 0 到 9 之 间的所有数字。
 
 ```
    $p = '/[\w\.\-]+@[a-z0-9\-]+\.(com|cn)/';
    $str = '我的邮箱是 code0809@163.com';
    preg_match($p, $str, $match);
    echo $match[0];
 ```
 
#### 贪婪模式与懒惰模式

正则表达式中每个元字符匹配一个字符，当使用 `+` 之后将会变的贪婪，它将匹配尽可能多的字符，但使用问号 `?` 字符时，它将尽可能少的匹配字符，即是**懒惰模式**。

> 懒惰模式：在可匹配与可不匹配的时候，优先不匹配。

```
    $p = '/\d?\-\d?/';
    $str = "我的电话是010-12345678";
    preg_match($p, $str, $match);
    echo $match[0]; //结果为:0-1
```

> 贪婪模式：在可匹配与可不匹配的时候，优先匹配。

```
    $p = '/\d+-\d+/';
    $str = '我的电话是 010-12345678';
    preg_match($p, $str, $match);
    echo $match[0];    // 结果为: 010-12345678
```

当我们确切的知道所匹配的字符长度的时候，可以使用 {} 指定匹配字符数：

```
    $p = '/\d{3}-\d{8}/';
    $str = "我的电话是010-12345678";
    preg_match($p, $str, $match);
    echo $match[0]; // 结果为: 010-12345678
```

#### 查找所有匹配结果

**preg_match** 只能匹配一次结果，但很多时候我们需要匹配所有的结果，**preg_match_all** 可以循环获取一个列表的匹配结果数组。

```
    $p = '|<>+>(.*?)</>+>|i';
    $str = "<b>example: </b><div align=left>this is a test</div>";
    preg_match_all($p, $str, $matches);
    print_r($matches);
```

使用 preg_match_all 匹配一个表格中的数据：

```
    $p = '/<tr><td>(.?)<\/td>\s<td>(.?)<\/td>\s<\/tr>/i';
    $str = '<table> <tr><td>Eric</td><td>25</td></tr> <tr> <td>John</td><td>26</td></tr> </table>';
    preg_match_all($p, $str, $matches);
    print_r($matches);
```

`$matches` 结果排序为 `$matches[0]` 保存完整模式的所有匹配，`$matches[1]` 保存第一个子组的所有匹配，以此类推。


#### 正则表达式的搜索和替换

正则表达式的搜索与替换在某些方面具有重要用途，比如调整目标字符串的格式，改变目标字符串中匹配字符串的顺序等。例如我们可以简单的调整字符串的日期格式：

```
    $string  = 'April 15, 2014';
    $pattern = '/(\w+) (\d+), (\d+)/i';
    $replacement = '$3, ${1} $2';
    echo preg_replace($pattern, $replacement, $string); // 结果为: 2014, April 15
```

其中 `${1}` 与 `$1` 的写法是等效的，表示第一个匹配的字串，`$2` 代表第二个 匹配的。 通过复杂的模式，我们可以更加精确的替换目标字符串的内容。

```
    $patterns = array ('/(19|20)(\d{2})-(\d{1,2})-(\d{1,2})/', '/^\s*{(\w+)}\s*=/');
    $replace = array ('\3/\4/\1\2', '$\1 =');    // \3等效于$3,\4等效于$4，依次类推
    echo preg_replace($patterns, $replace, '{startDate} = 1999-5-27');
    // 结果为: $startDate = 5/27/1999
```

详细解释下结果：`(19|20)` 表示取 `19` 或者 `20` 中任意一个数字，`(\d{2})` 表 示两个数字，`(\d{1,2})` 表示 `1` 个或 `2` 个数字，`(\d{1,2})` 表示 `1` 个或 `2` 个数 字。

`^\s*{(\w+)\s*=}` 表示以任意空格开头的，并且包含在 `{}` 中的字符， 并且以任意空格结尾的，最后有个 `=` 号的。

##### 查找并替换举例

```
    // 将目标字符串 $str 中的文件名替换后增加 em 标签
    $str = '主要有以下几个文件:index.php, style.css, common.js';
    $p='/\w+\.\w+/i';
    $str=preg_replace($p,'<em>$0</em>',$str);
    echo $str;
```

#### 用正则替换来去掉多余的空格与字符

```
    $str = 'one two';
    $str = preg_replace('/\s+/', ' ', $str);
    echo $str;    // 结果改变为'one two'
```

#### 用户信息验证

```
    <?php
    $user = array(
    	'name'   => 'spark1985',
    	'email'  => 'spark@imooc.com',
        'mobile' => '13312345678',
    );
    
    // 进行一般性验证
    if (empty($user)) {
    	die('用户信息不能为空');
    }
    if (mb_strlen($user['name']) < 6) {
      die('用户名长度最少为6位');
    }
    
    // 用户名必须为字母、数字与下划线
    if (!preg_match('/^\w+$/i', $user['name'])) {
    	die('用户名不合法');
    }
    
    // 验证邮箱格式是否正确
    if (!preg_match('/^[\w\.]+@\w+\.\w+$/i', $user['email'])) {
    	die('邮箱不合法');
    }
    
    // 手机号必须为 11 位数字 且为 1 开头
    if (!preg_match('/^1\d{10}$/i', $user['mobile'])) {
    	die('手机号不合法');
    }
    
    echo '用户信息验证成功';
```

#### 然后 PHP 中不要优先使用正则

正则的缺点：
    
 - 逻辑复杂，易出错
 - 效率低
 
> 能用 PHP 自身字符串 API 完成的，就不要使用正则。
 
## 信息加密技术分类

#### 单项散列加密

单项散列加密是指通过对不同输入长度的信息进行计算，得到固定长度的输出，输入相同计算结果相同。

但是，这个散列计算过程是单向的，即不能对固定长度的输出进行计算反过来的到原始输入信息。

此类算法的流程只有一步：`明文+单向散列算法+salt = 得到密文`。（不可逆）

> 盐（Salt），在密码学中，是指通过在密码任意固定位置插入特定的字符串，让散列后的结果和使用原始密码的散列结果不相符，这种过程称之为“加盐”。

#### 对称散列加密

对称加密是指加密和解密使用的密钥是同一个，或者可以互相推算。
 
 - 明文+加密算法+salt = 密文
 
 - 密文+解密算法+密钥 = 明文

#### 非对称散列加密

非对称加密和解密使用的密钥不是同一个，而是由算法生成的一对。

其中一个对外公开，被称作公钥，另一个私钥，只有所有者知道。

 - 明文+加密算法+加密密钥 = 密文
 
 - 密文+解密算法+密钥 = 明文

> 数字签名就是一种非对称散列加密技术常见的应用。

### PHP 内置的编码/加密/hash API

#### md5

md5 是简单单向加密过程，但是也可以通过字典和彩虹表来查询（猜测）获得( 通过遍历 事先保存过的一些常见的加密后的字符串，按查找的方式破解)。

因此，可以通过多次 md5() 加密来获得不常见的加密字符串在一定程度上提高难度：`md5(md5($str, true))`。

> 当第二个参数为 true 时，返回的是 16 字节的原始二进制格式。

#### crypt

`string crypt(string $str, [string $salt])`

返回基于标准 UNIX DES 算法或系统上其它可用的替代算法的散列字符串。

`$salt` 就是所谓的盐值，也就是干扰串。有效长度只能有 2 个。

#### sha1

和 md5() 使用相似，sha1 也是单项不可逆加密算法。（都可以通过碰撞穷举来获得原始字符串）

#### url 编码

 - `urlencode()`
 
 除了 `-/_/.` 之外的所有非字母和数字都将被替换成百分号 `%` 后跟两位十六进制数，空格则编码为 `+`。
 
 可以编码一些特殊字符然后用于传递 很多搜索引擎采用了这种“加密”方式。
 
 - `rawurlencode()`
 
 按照 RFC1738 对 URL 进行编码。和 urlencode() 的唯一区别就是，空格会被编码为 `%20`。
 
#### base64 编码

base64 最早用于电子邮件，因为早期的邮件网关只能识别 ASCII 码，其他字符将会被过滤。

把非 ASCII 字符( 中文/图片 )使用 base64 转换成 ASCII 字符传输，接收到后在进行转码即可，base64 适合在在 HTTP/MIME 协议中快速传输数据。

base64 也可以被用于编码/解码二进制数据(图片等)，严格地讲，base64 只是一种编码方式，而不是加密算法。

##### base64 保存一张图片

```
    $img = base64_encode(file_get_contents('a.jpg'));
    // #1
    header('Content-Type: image/jpeg');
    echo base64_decode($img);
    
    // #2
    $data = <<< IMG
      <img src="data:;base64, {$img}">
      <img src="data:image/jpeg;base64, {$img}">
    IMG;
    
    echo $data;
```


## 数制

### 补码

正数的原码和补码相同。

负数的补码 = 原码的反码+1。

补码给计算机看，原码给人看。


### 位运算

 - 算术左移
 低位溢出符号位不变，用符号位补溢出的高位。

 - 算术右移
 相当于每移一位乘以2 符号位不变，低位补零。

## 小算法

### 打印金字塔

 - 直角三角形
 
 ```
    /* 直角三角形 */
    # 外层的 for 循环控制层数
    for ($i=1; $i<=10; ++$i){
    	# 内层控制每层 * 号的个数
    	for ($j=1; $j<=$i; ++$j)
    		echo '*';
    	echo "\n";
    }
 ```
 
 - 实心金字塔
 
 ```
    /* 实心金字塔 */
    for ($i=1; $i<=5; ++$i) {
    	# 在打印 * 号前先打印空格
    	for ($k=1; $k<=5-$i; ++$k)
    		echo ' ';
    	for ($j=1; $j<=($i-1)*2+1; ++$j)
    		echo '*';
    	echo "\n";
    }
 ```
 
 - 空心金字塔

 ```
    /* 空心金字塔 */
    # Tips => 在给每行输出 `*` 号的时候 要判断改输出 `*` 号还是空格
    $n = 10;    // 金字塔层数
    for ($i=1; $i<=$n; ++$i) {
    	# 在打印 * 号前先打印空格
    	for ($k=1; $k<=$n-$i; ++$k)
    		echo ' ';
    	# 内层控制每层 * 号的个数
    	for($j=1; $j<=($i-1)*2+1; ++$j){
    	# 如果是第一层和最后一层 只输出 * 号
    		echo ($i==1 || $i==$n)
    			? '*' : (
    				($j==1 || $j==($i-1)*2+1)
    					? '*' : ' '
    			);
    	}
    	echo "\n";
    }
    // OR
    // 打印空心三角形
    function printOpenTriangle($n = 10) {
        for ($i=0; $i<$n; ++$i) {
            // print space - step 1
            for ($j=0; $j<$n-$i; ++$j) {
                echo ' ';
            }
            // print asterisk
            if ($i==0 or $i==$n-1) {
                for ($k=0; $k<=2*$i; ++$k) {
                    echo '*';
                }
            } else {
                echo '*';
                // print space - step 2
                for ($m=0; $m<=2*$i-2; ++$m) {
                    echo ' ';
                }
                echo '*';
            }
            // print newline
            echo PHP_EOL;
        }
    }
 ```
 
### 打印九九乘法表
    
```
    /* 99乘法表 */
    for ($i=1; $i<=9; ++$i) {
    	for ($j=1; $j<=$i; ++$j)
    		echo $i, 'x', $j, '=', $i*$j, ' ';
    	echo "\n";
    }
```


## 编程经验
 
### 如何看懂一段代码？

#### 郝斌
 
 - 每条语句的功能
 - 程序流程
 - 试数

#### 韩顺平

 - 代码执行顺序图
 - 内存变化图
 - 界面变化图
 - 程序流程图
 - 语句，真假，条件选择等

### 如何理解递归？

#### 从堆栈的角度

一个函数执行对应一个独立的栈空间，不管是不是同一个函数。

递归调用时，将一层层开辟堆栈，到底了后再一层层返回值。


#### 从内存的角度

变量按地址思考就不会混淆，`地址是变量内容的直接索引` 。


### 写代码的步骤？

 - 明确「需求」
 - 弄懂自己「要做什么」
 - 清晰「流程」
 
### 实战经验

 - 死去活来，先写死的测试功能再写活的完善界面和功能。
 - 数据库是项目的核心 写操作的设计的开始
 - 面向接口开发，如果有依赖，则优先制定出依赖间的接口 比如：数据存储格式、JSON API 返回格式


## FAQ

### override / overwrite / overload 的区别？

#### Overload 重载

在 C++ 程序中，可以将语义、功能相似的几个函数用同一个名字表示，但参数不同(包括类型、顺序不同)，即函数重载。

 - 相同的范围(在同一个类中)
 - 函数名字相同
 - 参数不同
 
请注意，重载解析中不考虑返回类型，而且在不同的作用域里声明的函数也不算是重载。


#### Override 覆盖

是指派生类函数覆盖基类函数，特征是：
 
 - 不同的范围(分别位于派生类与基类)
 - 函数名字相同
 - 参数相同
 - 基类函数必须有virtual 关键字
 
#### Overwrite 重写

是指派生类的函数屏蔽了与其同名的基类函数，规则如下：

 - 如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无 virtual 关键字，基类的函数将被隐藏。
 - 如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有 virtual 关键字。 此时，基类的函数被隐藏(注意别与覆盖混淆)。


### self 和 static

两者的差别主要体现在有继承关系的类之间。

简单下个结论：
 
 - 当在类中使用 `self::` 时，无论是否有继承关系，最终引用到的都是这个 self 单词所在的类。
 - 当在类中使用 `static::` 时，如果有继承关系，则最终引用到的是子类。
 
```
    class A {
        public static function get_A() {
            // return new self();
          	return __CLASS__;
        }
        public static function get_me() {
            return new static();
        }
    }
    class B extends A {}
    echo get_class(B::get_A());  // A
    echo get_class(B::get_me()); // B
    echo get_class(A::get_me()); // A
```

> suggestion：[后期静态绑定( PHP 5.3.0+)](http://php.net/manual/zh/language.oop5.late-static-bindings.php)。
>  
>  后期静态绑定本想通过引入一个新的关键字表示运行时最初调用的类来绕过限制。简单地说，这个关键字能够让你在上述例子中调用 B::get_me() 时引用的类是 B 而不是 A。最终决定不引入新的关键字，而是使用已经预留的 static 关键字。

#### static 其实和任何类对象没有必然的联系

> STATIC variable are not associated to any particular instance/object of a class. Hence you modify the variable with Parent Class reference or Child Class reference, the same copy gets modified.
>  
>  Hence apart from understanding Public Static as Global, Please understand it as not associated to any particular instance, hence with any class hierarchy reference you update a static variable , same memory location gets updated.

### 为什么可以使用 `::` 调用非静态方法

> This is actually an error of some kind if you have strict error reporting on, but not otherwise.

除此之外，PHP 还有其他怪癖行为（比如对象可以使用 -> 调用静态方法），因 PHP 语法松散而保留。但是，请避免使用它们。

## 参考

 - [PHP 笔记汇总](http://blog.cjli.info/2015/12/12/PHP-Notes/)
 - [What is the difference between self::$bar and static::$bar in PHP? [duplicate]](https://stackoverflow.com/questions/11710099/what-is-the-difference-between-selfbar-and-staticbar-in-php)
 - [php static property](https://stackoverflow.com/questions/14930046/php-static-property)
 - [Calling non static method with “::”](https://stackoverflow.com/questions/3754786/calling-non-static-method-with-double-colon)
 - [PHP: What if I call a static method in non-static way](https://stackoverflow.com/questions/12874376/php-what-if-i-call-a-static-method-in-non-static-way)
 - [when using self, parent, static and how?](https://stackoverflow.com/questions/10504129/when-using-self-parent-static-and-how)