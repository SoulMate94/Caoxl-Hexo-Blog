---
title: 整理有道云笔记
date: 2018-08-14 10:02:10
categories: Caoxl
tags: [Caoxl]
---

> 整理原来写在有道云上的一些资料笔记, 做初次整理,后续再一一对号入座

<!-- more -->

# PHP杂的识

## PHP 的一些高效写法

```    
    1.用单引号代替双引号来包含字符串，这样做会更快一些。因为PHP会在双引号包围的字符串中搜寻变量，单引号则不会，注意：只有echo能这么做，它是一种可以把多个字符串当作参数的“函数”（译注：PHP手册中说echo是语言结构，不是真正的函数，故把函数加上了双引号）。
    
    2.如果能将类的方法定义成static，就尽量定义成static，它的速度会提升将近4倍。
    
    3.$row['id'] 的速度是$row[id]的7倍。
    
    4.echo 比 print 快，并且使用echo的多重参数（译注：指用逗号而不是句点）代替字符串连接，比如echo $str1,$str2
    
    5.在执行for循环之前确定最大循环数，不要每循环一次都计算最大值，最好运用foreach代替
    
    6.注销那些不用的变量尤其是大数组，以便释放内存
    
    7.尽量避免使用__get，__set，__autoload
    
    8.require_once()代价昂贵
    
    9.include文件时尽量使用绝对路径，因为它避免了PHP去include_path里查找文件的速度，解析操作系统路径所需的时间会更少
    
    10.如果你想知道脚本开始执行（译注：即服务器端收到客户端请求）的时刻，使用$_SERVER['REQUEST_TIME']要好于time()
    
    11.函数代替正则表达式完成相同功能。
    
    12.str_replace函数比preg_replace函数快，但strtr函数的效率是str_replace函数的四倍
    
    13.如果一个字符串替换函数，可接受数组或字符作为参数，并且参数长度不太长，那么可以考虑额外写一段替换代码，使得每次传递参数是一个字符，而不是只写一行代码接受数组作为查询和替换的参数。
    
    14.使用选择分支语句（译注：即switch case）好于使用多个if，else if语句
    
    15.用@屏蔽错误消息的做法非常低效，极其低效
    
    16.打开apache的mod_deflate模块，可以提高网页的浏览速度
    
    17.数据库连接当使用完毕时应关掉，不要用长连接
    
    18.错误消息代价昂贵
    
    19.在方法中递增局部变量，速度是最快的。几乎与在函数中调用局部变量的速度相当
    
    20.递增一个全局变量要比递增一个局部变量慢2倍
    
    21.递增一个对象属性（如：$this->prop++）要比递增一个局部变量慢3倍。
    
    22.递增一个未预定义的局部变量要比递增一个预定义的局部变量慢9至10倍
    
    23.仅定义一个局部变量而没在函数中调用它，同样会减慢速度（其程度相当于递增一个局部变量）。PHP大概会检查看是否存在全局变量。
    
    24.方法调用看来与类中定义的方法的数量无关，因为我（在测试方法之前和之后都）添加了10个方法，但性能上没有变化
    
    25.派生类中的方法运行起来要快于在基类中定义的同样的方法
    
    26.调用带有一个参数的空函数，其花费的时间相当于执行7至8次的局部变量递增操作。类似的方法调用所花费的时间接近于15次的局部变量递增操作。
    
    27.Apache解析一个PHP脚本的时间要比解析一个静态HTML页面慢2至10倍。尽量多用静态HTML页面，少用脚本
    
    28.除非脚本可以缓存，否则每次调用时都会重新编译一次。引入一套PHP缓存机制通常可以提升25%至100%的性能，以免除编译开销。
    
    29.尽量做缓存，可使用memcached。memcached是一款高性能的内存对象缓存系统，可用来加速动态Web应用程序，减轻数据库负载。对运算码 (OP code)的缓存很有用，使得脚本不必为每个请求做重新编译。
    
    30.当操作字符串并需要检验其长度是否满足某种要求时，你想当然地会使用strlen()函数。此函数执行起来相当快，因为它不做任何计算，只返回在zval 结构（C的内置数据结构，用于存储PHP变量）中存储的已知字符串长度。但是，由于strlen()是函数，多多少少会有些慢，因为函数调用会经过诸多步骤，如字母小写化（译注：指函数名小写化，PHP不区分函数名大小写）、哈希查找，会跟随被调用的函数一起执行。在某些情况下，你可以使用isset() 技巧加速执行你的代码。
    （举例如下）
    if (strlen($foo) < 5) { echo “Foo is too short” }
    （与下面的技巧做比较）
    if (!isset($foo{5})) { echo “Foo is too short” }    
    调用isset()恰巧比strlen()快，因为与后者不同的是，isset()作为一种语言结构，意味着它的执行不需要函数查找和字母小写化。也就是说，实际上在检验字符串长度的顶层代码中你没有花太多开销。
    
    31.当执行变量$i的递增或递减时，$i++会比++$i慢一些。这种差异是PHP特有的，并不适用于其他语言，所以请不要修改你的C或Java代码并指望它们能立即变快，没用的。++$i更快是因为它只需要3条指令(opcodes)，$i++则需要4条指令。后置递增实际上会产生一个临时变量，这个临时变量随后被递增。而前置递增直接在原值上递增。这是最优化处理的一种，正如Zend的PHP优化器所作的那样。牢记这个优化处理不失为一个好主意，因为并不是所有的指令优化器都会做同样的优化处理，并且存在大量没有装配指令优化器的互联网服务提供商（ISPs）和服务器。
    
    32.并不是事必面向对象(OOP)，面向对象往往开销很大，每个方法和对象调用都会消耗很多内存
    
    33.并非要用类实现所有的数据结构，数组也很有用
    
    34.不要把方法细分得过多，仔细想想你真正打算重用的是哪些代码
    
    35.当你需要时，你总能把代码分解成方法。
    
    36.尽量采用大量的PHP内置函数。
    
    37.如果在代码中存在大量耗时的函数，你可以考虑用C扩展的方式实现它们
    
    38.评估检验(profile)你的代码。检验器会告诉你，代码的哪些部分消耗了多少时间。Xdebug调试器包含了检验程序，评估检验总体上可以显示出代码的瓶颈。
    
    39.mod_zip可作为Apache模块，用来即时压缩你的数据，并可让数据传输量降低80%。
    
    40.在可以用file_get_contents替代file、fopen、feof、fgets等系列方法的情况下，尽量用file_get_contents，因为他的效率高得多！但是要注意file_get_contents在打开一个URL文件时候的PHP版本问题；
    
    41.尽量的少进行文件操作，虽然PHP的文件操作效率也不低的；
    
    42.优化Select SQL语句，在可能的情况下尽量少的进行Insert、Update操作(在update上，我被恶批过)
    
    43.尽可能的使用PHP内部函数（但是我却为了找个PHP里面不存在的函数，浪费了本可以写出一个自定义函数的时间，经验问题啊！）；
    
    44.循环内部不要声明变量，尤其是大变量：对象(这好像不只是PHP里面要注意的问题吧？)
    
    45.多维数组尽量不要循环嵌套赋值；
    
    46.在可以用PHP内部字符串操作函数的情况下，不要用正则表达式
    
    47.foreach效率更高，尽量用foreach代替while和for循环
    
    48.用单引号替代双引号引用字符串；
    
    49.“用i+=1代替i=i+1。符合c/c++的习惯，效率还高”
    
    50.对global变量，应该用完就unset()掉
```

## 命名规则

```
    常量名 类常量建议全大写，单词间用下划线分隔    // MIN_WIDTH
    变量名建议用下划线方式分隔            // $var_name
    函数名建议用驼峰命名法                // varName
    定界符建议全大写                 // <<<DING, <<<'DING'
    文件名建议全小写和下划线、数字        // func_name.php
    私有属性名、方法名建议加下划线        // private $_name _func
    接口名建议加I_                    // interface I_Name
```

## 配置选项

```
    set_time_limit($seconds) //设置脚本最大执行时间(默认30秒)，0表示不限制
    ini_get($varname) //获取一个配置选项的值
    ini_set($varname, $newvalue) //为一个配置选项设置值
    extension_loaded($ext_name) //检测一个扩展是否已经加载
    get_extension_funcs($ext_name) //返回模块函数名的数组
```

## 命令行CLI

```
    //显示帮助信息
    php -h
    //解析并运行-f选项给定的文件名
    php [-f] <file> [--] [args...]
    //在命令行内运行单行PHP代码
    php [options] -r <code> [--] [args...]
    无需加上PHP的起始和结束标识符，否则将会导致语法解析错误
    //调用phpinfo()函数并显示出结果
    php -i/--info
    //检查PHP语法
    php -l/--syntax-check
    //打印出内置以及已加载的PHP及Zend模块
    php -m/--modules
    //将PHP，PHP SAPI和Zend的版本信息写入标准输出
    php -v/--version
    
    //参数接收
    $argv    传递给脚本的参数数组
        第一个参数总是当前脚本的文件名，因此$argv[0]就是脚本文件名
    $argc    传递给脚本的参数数目
        脚本的文件名总是作为参数传递给当前脚本，因此$argc的最小值为1
    包含当运行于命令行下时传递给当前脚本的参数的数组
    此两个变量仅在register_argc_argv打开时可用
```

## 类和对象

```
    # 成员：
        类成员：类常量、静态属性、静态方法
        对象成员：非静态属性、非静态方法
        # 除此外，类不能包含任何其他东西！！！
    
    # 类名、方法名、属性名均不区分大小写
    # $this代表本对象，self代表本类，parent代表父类
    # 类和函数均可被事先编译（仅作为最外层时）
    # 类的定义必须在单一的PHP区块内，不能被多个PHP标签分割
```

```
    / 构造方法
    - 具有构造函数的类会在每次创建新对象时先调用此方法
    void __construct([ mixed $args [, $... ]] )
    - 构造方法所需参数由new实例化对象时，给类增加参数值。
    - 构造方法也可以被手动调用。
    - 5.3.3版本以前，支持于类名同名的方法作为构造方法。
    - 两种冲突时，__construct 优先
    
    // 析构方法
    - 析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。
    void __destruct( void )
    # 作用：释放对象所占用的资源
    # 调用的时机 
        - 脚本结束时所有资源均被释放，包括对象
        - 手动删除对象时
        - 保存对象的变量被赋予新值时(任何值，包括null)
        - 在使用exit()终止脚本运行时也会被调用
    
    // 静态成员(static关键字)
        - 声明类成员或方法为static，就可以不实例化类而直接访问。
        - 静态成员（属性或方法）均属于类，故不能通过$this或->访问。
        - 静态成员是所有对象共享，属于类。
        - 静态成员用类调用，非静态成员用对象调用。
    # 静态属性
        - 静态属性不可以由对象通过->操作符来访问。
        - 静态属性只能被初始化为一个字符值或一个常量，不能使用表达式。 所以你可以把静态属性初始化为整型或数组，但不能指向另一个变量或函数返回值，也不能指向一个对象。
    # 静态方法
        - 由于静态方法不需要通过对象即可调用，所以伪变量$this在静态方法中不可用。
        - 用::方式调用一个非静态方法会导致一个E_STRICT级别的错误。
    
    // 访问解析操作符(::)
        - 可以用于访问静态成员、方法和常量，还可以用于覆盖类中的成员和方法。 
        - 当在类的外部访问这些静态成员、方法和常量时，必须使用类的名字。 
        - self 和 parent 这两个特殊的关键字是用于在类的内部对成员或方法进行访问的。
    
    // 访问辨析
    - 对象成员，内部通过$this指定，外部通过对象名指定，均用->访问，访问属性时不需加$。
        对象名->属性名    对象名->方法名()    $this->属性名        $this->方法名()
    - 类成员，内部通过self或parent指定，外部通过类名指定，均用::访问，访问属性时需加$。
        类名::$属性名    类名::方法名()        self::$属性名        self::方法名()
    - 特殊：也可以通过对象访问类成员。（不建议）
        对象名::$类属性名    $this::$类属性名    对象名::$类方法名()    $this::类方法名()
    # 对象成员访问用->，类成员访问用::
    
    - 无论是静态方法还是非静态方法，均可通过类或对象进行访问。
    - 静态属性通过类访问，静态方法通过对象访问。
    - 只有使用对象调用非静态方法时，$this才可以使用！
    - 静态方法不可使用$this。
    - 类可以调用对象方法，但注意方法内不能有$this。
    - 非静态方法可以调用静态属性或静态方法，反之不可以。
    
    // 类常量
    - 常量的值将始终保持不变。
    - 在定义和使用常量的时候不需要使用$符号。
    - 常量的值必须是一个定值，不能是变量，类属性或其它操作（如函数调用）的结果。
    # 定义：const 常量名 = 常量值;
    - 不需要加public等访问修饰限定符
    - 类常量属于类，使用类访问，类名::类常量 或 self::类常量
    
    // 自动加载对象
    - 在试图使用尚未被定义的类时自动调用 __autoload 函数
    - 自动加载使用到的类名文件（根据类名找相应名称的文件，故需类名与类文件名一致）
    - 每个需要加载类的文件都需要存在__autoload函数
    - 将__autoload函数写入单独的文件，每个需要用到类的文件再require该函数文件
    - __autoload 参数是类名
    function __autoload($class_name) {
        require_once $_SERVER["DOCUMENT_ROOT"] . "/class/$class_name.php";
    }
        // $_SERVER["DOCUMENT_ROOT"] 当前运行脚本所在的文档根目录
    - 可以通过类名，来推导出类所在的文件名！
    - 如果一个项目存在多个自动加载函数时，定义一个可以完成加载的普通函数，并在函数之前使用spl_autoload_register注册该函数。
    # spl_autoload_register
    - 注册__autoload()函数
    bool spl_autoload_register ([ callback $autoload_function ] )
    - 可以注册多个自动加载函数，先注册的先执行
    - 一旦注册自动加载函数，__autoload就失效。
    - 注册函数时，参数为函数名（注意加引号）；注册方法时，参数为数组
    # 注册类或对象的方法为自动加载方法时，参数需为数组：
    spl_autoload_register(array(__CLASS__, '__autoload'));
    __CLASS__表示当前类名，若是对象可用$this，详细见手册
    
    // 序列化（串行化）
    # 数据传输均是字符串类型
    # 除了资源类型，均可序列化
    # 序列化在存放数据时，会存放数据本身，也会存放数据类型
    作用：1.在网络传输数据时；2.为了将数组或对象放在磁盘时
    # 序列化
    serialize        产生一个可存储的值的表示
    string serialize ( mixed $value )
    - 返回字符串，此字符串包含了表示value的字节流，可以存储于任何地方。
    - 有利于存储或传递 PHP 的值，同时不丢失其类型和结构。
    # 反序列化
    unserialize        从已存储的表示中创建PHP的值
    mixed unserialize ( string $str [, string $callback ] )
    - 对单一的已序列化的变量进行操作，将其转换回PHP的值。
    
    
    # 文件的读写操作
    - file_put_contents        将一个字符串写入文件
    int file_put_contents($file, $data [,$flags])
        $flags：FILE_USE_INCLUDE_PATH(覆盖)，FILE_APPEND(追加)
    - file_get_contents        将整个文件读入一个字符串
    string file_get_contents($file [, bool $use_include_path [,int $offset [,int $maxlen]]])
    
    # 对象序列化
    - 只能序列化对象内部的数据，即非静态属性。
    # 需在反序列化对象之前加载类，也可以触发自动加载机制。
    
    __sleep        序列化需序列化的属性。
            - 提交未提交的数据，或类似的清理操作，部分串行化对象。
            - 返回一个包含对象中所有应被序列化的变量名称的数组
    __wakeup    反序列化时，预先准备对象需要的资源
            - 重新建立数据库连接，或执行其它初始化操作
        public function __sleep() {
            return array('server', 'username', 'password', 'db');
        }
        public function __wakeup() {
            $this->connect();
        }
    
    // 对象继承
    class 子类名 extends 父类 {}
    如果一个对象是子类的对象，那么同时也是父类的对象。
    单继承：一个类只能继承一个父类，不能同时继承多个类。但一个父类可以被多个子类继承。
    
    instanceof    判断某对象是否为某类的对象
        对象名 instanceof 类名
    
    // 访问控制
    public        公有的（继承链、本类、外部均可访问）
    protected    保护的（仅继承链、本类可访问）
    private        私有的（仅本类可访问）
    根据成员定义位置、访问位置判断。
    # 兼容性问题
    - 声明属性时，var关键字声明的默认为public权限
    - 声明方法时，省略访问修饰符，默认为public权限
    
    // 重写 override
    $this代表本对象，被谁调用，就代表哪个对象。
    - 继承时，子类成员名于父类成员名发生冲突，则子类成员会重写父类成员。
    - 属性和方法均可被子类重写。
    - 当父类的方法或属性已经不满足子类的需求，则需要重写。
    - 也可能因为命名不规范导致重写。
    
    私有属性不能被重写，每个私有属性都会被记录。在记录属性名的同时，还会记录类。
    
    如果有内置函数被重写，则可调用父类方法。如调用父类构造方法parent::__construct()
    
    # 重写限制
    访问限制：
        子类的成员的访问控制必须相等或弱于父类。
    方法参数限制：
        参数数量必须相同，参数名可不同。
    
    # $this确定原则
    $this为调用该方法的对象，表示该方法的执行环境对象。
        - 对象调用
        - 环境的传递。如果当前调用时，不能确定$this的值(静态调用)，此时静态调用所处对象环境会传递到被调用的方法内。
    $this并非永远代表本对象，而是由方法的执行环境决定。
    
    # final
    如果父类中的方法被声明为final，则子类无法覆盖（重写）该方法。
    如果一个类被声明为final，则不能被继承。
    但加有final关键字的类依旧能被实例化！
    # 抽象类
    关键字：abstract
    抽象类不能直接被实例化，必须先继承该抽象类，然后再实例化子类。
    抽象类中至少要包含一个抽象方法。非抽象类不能包含抽象方法。
    如果类方法被声明为抽象的，那么其中就不能包括具体的功能实现。抽象方法不能包含大括号及方法体。
    继承一个抽象类的时候，子类必须实现抽象类中的所有抽象方法。
        即，子类必须重写抽象父类中的所有抽象方法。
    另外，这些方法的可见性必须和抽象类中一样（或者更为宽松）。
        即，如果抽象类中某个抽象方法被声明为protected，那么子类中实现的方法就应该声明为protected或者public，而不能定义为private。
    - 抽象类的子类中的普通方法执行方式和其他类相同。
    - 作用：
        1. 继承，为扩展类，统一公共操作。
        2. 限制结构（规范）。规范子类的结构。
    
    // 接口
    关键字：interface
    - 对象提供的与对象交互的方式就是接口。
    - 使用接口可以指定某个类必须实现哪些方法，但不需要定义这些方法的具体内容。
    - 通过interface来定义一个接口，就像定义一个标准的类一样，但其中定义所有的方法都是空的。 
    - 接口中定义的所有属性和方法都必须是public，可省略public关键字。
    - 接口中也可以定义常量(const)。接口常量和类常量的使用完全相同。
        可以用::访问。接口名::常量名，实现类::常量名。
        它们都是定值，可以被子类或子接口使用，但不能修改。
    - 接口不能定义属性！
    # 定义接口
    interface 接口名 {
        接口内容（公共方法声明的集合）
    }
    # 接口实现
    - 要实现一个接口，可以使用implements操作符。
    - 类中必须实现接口中定义的所有方法，否则会报一个fatal错误。
    - 如果要实现多个接口，可以用逗号来分隔多个接口的名称。
    - 实现多个接口时，接口中的方法不能有重名。
    - 接口也可以继承，通过使用extends操作符。
    class 类名 implements 接口名 {
        接口方法的实现
    }
    # 注意
        1. 类与抽象类之间是继承关系，类与接口之间是实现关系。
        2. 类与抽象类是单继承，类与接口是多实现。
        3. 接口不是类，限制类的结构。
        4. 接口与接口之间是多继承。用extends关键字。
            interface I_C extends I_A, I_B {}
    
    // 静态延迟绑定
    self::，代表本类(当前代码所在类)
        永远代表本类，因为在类编译时已经被确定。
        即，子类调用父类方法，self却不代表调用的子类。
    static::，代表本类(调用该方法的类)
        用于在继承范围内引用静态调用的类。
        运行时，才确定代表的类。
        static::不再被解析为定义当前方法所在的类，而是在实际运行时计算的。
    
    // 对象的遍历（迭代）
    - 对象通过属性保存数据，故遍历对象的属性。
    - foreach语言结构，获得属性名和属性值。
        foreach ($obj as $p_name => $p_value) {}
    # 自定义遍历(迭代器Iterator)
    Iterator - 可在内部迭代自己的外部迭代器或类的接口
    Iterator::current    — 返回当前元素
    Iterator::key        — 返回当前元素的键
    Iterator::next        — 向前移动到下一个元素
    Iterator::rewind    — 返回到迭代器的第一个元素
    Iterator::valid        — 检查当前位置是否有效
    
    # 对象的克隆
    //对象之间的传值是[引用]传递。
    克隆：新对象 = clone 旧对象
        - 所有的引用属性仍然会是一个指向原来的变量的引用。 
    __clone()方法在对象被克隆时自动调用。
    注意：构造方法对应实例化(new)，克隆方法对应克隆(clone)。
    
    // 单例模式
    #三私一公
    单例模式（Singleton）用于为一个类生成一个唯一的对象。最常用的地方是数据库连接。使用单例模式生成一个对象后，该对象可以被其它众多对象所使用。
    # 防止一个类被实例化多次
    class MySQLDB {
        private static $instance = null; // 存类实例在此属性中
        // 构造方法声明为private，防止直接创建对象
        private function __construct() {}
        public static function getInstance() {
            if(! self::$instance instanceof static) {
                self::$instance = new static;
            }
            return self::$instance;
        }
        private function __clone() {} // 阻止用户复制对象实例
    }
    
    // 魔术方法
    __construct        构造方法
    __destruct        析构方法
    __clone            克隆对象
    __sleep            序列化对象
    __wakeup        反序列化对象
    __autoload        自动加载，使用类但未找到时
    
    __toString        对象被当作字符串使用时
    __invoke        当尝试以调用函数的方式调用一个对象时
    
    # 重载 overload
    指动态地"创建"类属性和方法
    用户可以自由的为对象添加额外的属性，该特性就是重载。
    所有的重载方法都必须被声明为public。
    当调用当前环境下未定义或不可见的类属性或方法时，重载方法会被调用。
    重载相关魔术方法的参数都不能通过引用传递。
    # 属性重载
    - 处理不可访问的属性
    属性重载只能在对象中进行。
    # 属性重载对于静态属性无效
    在静态方法中，这些魔术方法将不会被调用。所以这些方法都不能被声明为static。
    __set        在给不可访问的属性赋值时
        public void __set(string $name, mixed $value)
        作用：批量管理私有属性，间接保护对象结构
    __get        读取不可访问的属性的值时
        public mixed __get(string $name)
    __isset        当对不可访问的属性调用isset()或empty()时
        public bool __isset(string $name)
    __unset        当对不可访问的属性调用unset()时
        public void __unset(string $name)
    # 方法重载
    - 处理不可访问的方法
    __call            当调用一个不可访问的非静态方法（如未定义，或者不可见）时自动被调用
            public mixed __call(string $name, array $arguments)
    __callStatic    当在调用一个不可访问的静态方法（如未定义，或者不可见）时自动被调用
            public static mixed __callStatic(string $name, array $arguments)
    # $name参数是要调用的方法名称。$arguments参数是一个数组，包含着要传递给方法的参数。
    
    // 类型约束
    函数的参数可以指定只能为对象或数组
    限定为对象则在形参前加类名，限定为数组则在形参前加array
    类型约束允许NULL值
    类型约束不只是用在类的成员方法里，也能使用在函数里。 
    
    // 三大特性
    封装：隐藏内部是吸纳，仅开发接口。
    继承：一个对象的成员被另一个对象所使用。语法上体现为代码的共用。
    多态：多种形态。
    
    // 类与对象·关键字
    this        代表本对象
    public        公有的（继承链、本类、外部均可访问）
    protected    保护的（仅继承链、本类可访问）
    private        私有的（仅本类可访问）
    parent::    代表父类
    self::        代表本类(当前代码所在类)
    static::    代表本类(调用该方法的类)
    static        静态成员（属性、方法），所有对象均可使用，外部也可直接使用或修改，静态方法不可访问非静态成员
    final        方法用final不可被子类重载，类用final不可被继承（方法、类）
    const        类常量（属性）
    abstract    抽象类
    interface    接口
    extends        类继承(子接口继承接口、其他普通类继承)
    implements    接口实现（类实现接口、抽象类实现借口）（对接口的实现和继承均可有多个）
    Iterator    内置接口（迭代）
    clone        克隆
    instance    实例
    instanceof    某对象是否属于某类
    
    /* 【类与对象相关函数】 */
    class_alias([$original [,$alias]])  给类取别名
    class_exists($class [,$autoload])   检查类是否已定义
    interface_exists($interface [,$autoload])   检查接口是否已被定义
    method_exists($obj, $method)检查类的方法是否存在
    property_exists($class, $property)  检查对象或类是否具有该属性
    get_declared_classes(void)  返回由已定义类的名字所组成的数组
    get_declared_interfaces(void)   返回一个数组包含所有已声明的接口
    get_class([$obj])       返回对象的类名
    get_parent_class([$obj])    返回对象或类的父类名
    get_class_methods($class)   返回由类的方法名组成的数组
    get_object_vars($obj)   返回由对象属性组成的关联数组
    get_class_vars($class)  返回由类的默认属性组成的数组
    is_a($obj, $class) 如果对象属于该类或该类是此对象的父类则返回TRUE
    is_subclass_of($obj, $class)    如果此对象是该类的子类，则返回TRUE
    get_object_vars($obj)   返回由对象属性组成的关联数组
    
    
    // 常用类
    # PHP手册 -> 预定义类
    Closure        闭包类，匿名函数对象的final类
    stdClass    标准类，通常用于对象类保存集合数据
    __PHP_Incomplete_Class        不完整类，当只有对象而没有找到类时，则该对象被认为是该类的对象
    Exception    异常类
    PDO            数据对象类
    
    // 魔术常量
    __DIR__            文件所在的目录
    __LINE__        文件中的当前行号 
    __FILE__        文件的完整路径（绝对路径）和文件名
    
    __CLASS__        类的名称
    __METHOD__        类的方法名，包含类名和方法名
    __FUNCTION__    函数名称，用在方法内只表示方法名
    
    // 反射机制 Reflection
    作用：1. 获取结构信息        2. 代理执行
    ReflectionClass 报告一个类的有关信息
    ReflectionMethod 报告一个方法的有关信息
    ReflectionClass::export    输出类结构报告
    # 代理执行
    实例化 ReflectionFunction 类的对象
        $f = new ReflectionFunction('func');    // func为函数func($p)
        $f->invoke('param');
```

## 网站并发

```
    测试工具：apache/bin/ab.exe
    用法：cmd{%apache-bin%}>ab.exe -n 执行访问次数 -c 用户并发数量 URL地址
    MPM(多路处理模块)：perfork(预处理模式), worker(工作者模式), winnt(Win系统)
    MPM配置：httpd-mpm.conf
    查看当前MPM模式：httpd –l    mpm_xxx.c中xxx表示当前模式类型
    httpd.conf配置(开启MPM)：#Include conf/extra/httpd-mpm.conf
    #参考配置
    #配置文件：extra/httpd-mpm.conf
    #mpm_winnt.c
    <IfModule mpm_winnt_module>
        ThreadsPerChild      1000   #中型网站1500-5500合理
        MaxRequestsPerChild  0
    </IfModule>
    #mpm_prefork.c
    <IfModule mpm_prefork_module>
        StartServers    5       #预先启动
        MinSpareServers 5
        MaxSpareServers 10      #最大空闲进程
        ServerLimit     1500    #用于修改apache编程参数
        MaxClients      1000    #最大并发数
        MaxRequestsPerChild 0   #一个进程对应的线程数，对worker更用
    </IfModule>
    #如果你的网站pv值上百万
    ServerLimit     2500   #用于修改apache编程参数
    MaxClients      2000   #最大并发数
```

## 预定义常量

```  
    PATH_SEPARATOR  //路径分隔符(Windows为分号，类Unix为冒号)
    DIRECTORY_SEPARATOR //目录分隔符
    PHP_EOL //当前系统的换行符
    PHP_VERSION //PHP版本号
    PHP_OS  //PHP服务操作系统
    PHP_SAPI    //用来判断是使用命令行还是浏览器执行的，如果 PHP_SAPI=='cli' 表示是在命令行下执行
    PHP_INT_MAX                    INT最大值，32位平台时值为2147483647
    PHP_INT_SIZE                   INT字长，32位平台时值为4（4字节）
    M_PI    //圆周率值
    M_E     //自然数
```

## 相对路径

```
    当前浏览器请求的哪个脚本，当前位置就是属于哪个脚本。
    ./file 和 file 都表示当前目录下的file文件
    file情况（嵌套载入文件时）：
    如果当前目录没找到该文件就在代码文件所在目录中继续找。
    如果当前目录找到有该文件，则不会再在代码文件所在目录去找也不会再加载。
    __DIR__     脚本文件所在目录
    __FILE__    脚本文件路径
    
    include_path    加载文件查找目录
        set_include_path()  设置include_path，可多个，用字符串作参数
        该函数设置的path只针对该当前文件有效
        该设置只针对查找未直接写文件路径方式有效
        设置新的include_path会覆盖原来的
    
        get_include_path()  获取当前include_path设置项，无参数
    
        路径分隔符，在Windows下是分号，在Linux下是冒号
        利用预定义常量 PATH_SEPARATOR 来获得当前的分隔符
    
    如果直接写文件名：
        1. include_path所设置的
        2. 当前目录
        3. 代码所在文件的目录
    如果文件名前带有路径，则会直接根据路径查找，include_path直接被忽略
```

## 变量函数

```
    get_defined_vars    //返回由所有已定义变量所组成的数组(包括环境变量、服务器变量和用户定义的变量)
```

## 可变标识符

```
    可变变量  $i = 3; $k = 'i'; echo $$k; //输出3
    可变函数  function func() {echo 'hello!';} $i = 'func'; $i(); //输出hello
    可变下标  $i = '1234'; $k = 3; echo $i[$k];   //输出4
    可变类名  class CLS{public $k = 'hello';} $i = 'CLS'; $j = new $i; echo $j->k;
    可变属性  class CLS{public $k = 'hello';} $i = 'k'; $j = new CLS; echo $j->$i;
    可变方法  class CLS{public function k(){echo 'hello';}} $i='k'; $j=new CLS; $j->$i();
```

## 终止和延迟脚本执行

```
    die / exit   终止
    return是终止所在脚本的执行
    die和exit会立即终止脚本执行
    die("到此为止");     该函数内的字符串可被输出
    sleep()  延迟(单位：秒)
        默认最多可延迟30秒，PHP配置可以修改 max_execution_time
        例：sleep(12);
    usleep()    以指定的微秒数延迟执行
    time_sleep_until    使脚本睡眠到指定的时间为止
```

## 定界符

```
    herodoc - 功能同双引号，能解析
    $str = <<<AAA
    字符串内容
    AAA;
    
    nowdoc - 功能同单引号，不能解析
    只在开始位置有单引号
    $str = <<<'AAA'
    字符串内容
    AAA;
```

## 静态化

```
    1. 页面URL长度不超过255字节
    2. meta信息尽量完整，keywords5个左右
    3. 前端不要使用框架
    4. 图片alt属性添加信息
    5. 静态页面不要带动态值
    
    <script type="text/javascript" language="javascript" src="url"></script>
    url可以是js/php/图片等，返回的数据替换<script>标签所在位置的内容！相当于简单的Ajax
```

## 常量相关

```
    /* 常量定义 */
    define(常量名, 常量值, [区分大小写参数])        //true表示不区分/false表示区分大小写
    const 常量名 = 常量值    // 新，建议
    常量名可以使用特殊字符
    constant($name)        // 获取常量名
                           // 例：echo constant('-_-');
    
    /* 常量相关函数 */
    defined
    get_defined_constants
    
    /* 预定义常量 */
    __FILE__            所在文件的绝对路径
    __LINE__            文件中的当前行号
    __DIR__            文件所在目录
    __FUNCTION__        函数名称
    __CLASS__            类的名称
    __METHOD__        类的方法名
    __NAMESPACE__        当前命名空间的名称
```

## 变量生命周期

1. 脚本结束时，全局变量消失
2. 函数执行完时，局部变量消失
3. 静态变量
    static关键字
        静态变量仅在局部函数域中存在，但当程序执行离开此作用域时，其值并不丢失。
        静态变量仅会被初始化一次，其他局部变量每次被调用时都会被重新赋值。
        static声明的静态变量的生命周期会被一直延续。

## 变量作用域

a. 全局变量和局部变量
    1) 作用域之间不重叠，即不同作用域的变量，之间不可访问
    2) 全局作用域  - 函数之外的区域
    3) 局部作用域  - 函数内的区域，每个函数都是一个独立的作用域

b. 超全局变量，既可以在全局也可在局部使用，仅能用系统自带的，均是数组变量。
    $GLOBALS    $_COOKIE    $_ENV       $_FILES $_GET
    $_POST      $_REQUEST   $_SERVER    $_SESSION
c. $GLOBALS
    1) 不能存在超全局变量，但可以有超全局的数据！
    2) 将需要的数据放到超全局变量的数组内，但统一使用$GLOBALS
    3) $GLOBALS 特征
        - 每个全局变量就是对应$GLOBALS内的一个元素！
            每当增加一个全局，则自动在$GLOBALS内增加一个同名元素！
            同理，每当增加元素，也会增加一个全局变量，一般在函数内增加
        - 做的任何修改，也会映射到另一个，包括更新和删除
            在函数内访问全局变量，只需使用$GLOBALS
        - 出现过的全局变量，就可以通过$GLOBALS这个数组取得
    4) PHP生命周期中，定义在函数体外部的所谓全局变量，函数内部是不能直接获得的
4) global关键字（不建议使用）
    将局部变量声明为同名全局变量的一个'引用'！相当于常量的引用传递
        global $var;    // $var = &$GLOBALS['var'];
        不同于$GLOBALS！！！
    global在函数产生一个指向函数外部变量的别名变量，而不是真正的函数外部变量。
    $GLOBALS确确实实调用是外部的变量，函数内外会始终保持一致。
    global的作用是定义全局变量，但是这个全局变量不是应用于整个网站，而是应用于当前页面，包括include或require的所有文件。
d. 
    1) 作用域只针对变量，对常量无效
    2) 被载入文件中定义的变量作用域取决于被载入的位置。
        函数外被载入就是全局，函数内被载入就是局部！


## 错误处理

解析错误、运行错误
//标准错误：
    级别、信息、文件、行号
    trigger_error   触发一个用户自定义的error/warning/notice错误信息

//php.ini配置，ini_set()
error_reporting         设置报告哪些级别的错误

```
    # 错误报告：显示到页面
        display_errors = On 是否显示错误报告
    # 错误日志：存放到文件
        log_errors = on     是否开启错误日志
        error_log           发送错误信息到错误日志文件
    - 错误报告和错误日志可同时启用！
```

自定义错误处理器
set_error_handler — 注册自定义错误处理器函数
- 自定义处理器函数包含4个参数，分别是级别、信息、文件、行号
- 开启自定义错误处理器，则系统内置的错误报告和错误日志则不会执行。
- 自定义错误处理器函数返回false，则自定义函数结束后系统内置的会继续执行。
- 用户定义的错误级别(E_USER_ERROR)，可以被自定义的错误处理器所捕获并继续执行。系统内置的错误，则脚本会立即停止。
restore_error_handler — 恢复预定义错误处理器函数
error_get_last — 获取最近的错误信息

//错误处理函数
debug_backtrace 产生一条回溯跟踪
    返回数组，包含的键值：function, line, file, class, object, type, args
debug_print_backtrace 打印一条回溯

//错误常量
手册>错误处理

#生产模式
关闭错误报告，记录错误日志。
#开发模式
关闭错误日志，开启错误报告。

//异常
面向对象语法中的错误处理方式。
一个异常就是一个包含当前异常信息的对象。
预定义异常类Exception及其扩展类。
#抛出异常
触发一个异常的错误
throw new UserException();
如果没有被捕获，则报告致命错误。
#监视异常
try {代码段}
#捕获异常
catch (UserException $obj) {代码段}
需要通过当前异常的类型匹配才可悲捕获。
#异常处理器
用以处理未被捕获的异常。
异常处理器函数与catch类似，参数也是含类型的对象。
set_exception_handler — 注册异常处理器函数
restore_exception_handler — 恢复预定义的异常处理器函数


#自定义异常
用户定义的异常类须继承自Exception类。

//异常相关属性
protected string $message 异常消息内容
protected int $code 异常代码
protected string $file 抛出异常的文件名
protected int $line 抛出异常在该文件中的行号
//异常相关方法
Exception::__construct — 异常构造函数
Exception::getMessage — 获取异常消息内容
Exception::getPrevious — 返回异常链中的前一个异常
Exception::getCode — 获取异常代码
Exception::getFile — 获取发生异常的程序文件名称
Exception::getLine — 获取发生异常的代码在文件中的行号
Exception::getTrace — 获取异常追踪信息
Exception::getTraceAsString — 获取字符串类型的异常追踪信息
Exception::__toString — 将异常对象转换为字符串
Exception::__clone — 异常克隆


## $_SERVER

```
   //示例URL：http://desktop/dir/demo.php?a=aaa&b=bbb
   PHP_SELF 当前执行脚本的文件名 // /dir/demo.php
   GATEWAY_INTERFACE 服务器使用的CGI规范的版本 // CGI/1.1
   SERVER_ADDR 当前运行脚本所在的服务器的IP地址 // 127.0.0.1
   SERVER_NAME 当前运行脚本所在的服务器的主机名 // desktop
   SERVER_SOFTWARE 服务器标识字符串 // Apache/2.2.22 (Win32) PHP/5.3.13
   SERVER_PROTOCOL 请求页面时通信协议的名称和版本 // HTTP/1.1
   REQUEST_METHOD 访问页面使用的请求方式 // GET
   REQUEST_TIME 请求开始时的时间戳 // 1386032633
   QUERY_STRING 查询字符串(参数) // a=aaa&b=bbb
   DOCUMENT_ROOT 当前运行脚本所在的文档根目录 // C:/Users/Administrator/Desktop
   HTTP_ACCEPT 当前请求头中Accept:项的内容 // text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
   HTTP_ACCEPT_CHARSET 当前请求头中Accept-Charset:项的内容 // UTF-8,*
   HTTP_ACCEPT_ENCODING 当前请求头中Accept-Encoding:项的内容 // gzip, deflate
   HTTP_ACCEPT_LANGUAGE 当前请求头中Accept-Language:项的内容 // zh-cn,zh;q=0.5
   HTTP_CONNECTION 当前请求头中Connection:项的内容 // keep-alive
   HTTP_HOST 当前请求头中Host:项的内容 // desktop
   HTTP_REFERER 引导用户代理到当前页的前一页的地址
   HTTP_USER_AGENT 当前请求头中User-Agent:项的内容 // Mozilla/5.0 (Windows NT 6.1; rv:2.0.1) Gecko/20100101 Firefox/4.0.1
   HTTPS 如果脚本是通过HTTPS协议被访问，则被设为一个非空的值
   REMOTE_ADDR 浏览当前页面的用户的IP地址 // 127.0.0.1
   REMOTE_HOST 浏览当前页面的用户的主机名
   REMOTE_PORT 用户机器上连接到Web服务器所使用的端口号 // 49197
   REMOTE_USER 经验证的用户
   REDIRECT_REMOTE_USER 验证的用户，如果请求已在内部重定向
   SCRIPT_FILENAME 当前执行脚本的绝对路径 // C:/Users/Administrator/Desktop/dir/demo.php
   SERVER_ADMIN 该值指明了Apache服务器配置文件中的SERVER_ADMIN参数 //admin@shocker.com
   SERVER_PORT Web服务器使用的端口 // 80
   SERVER_SIGNATURE 包含了服务器版本和虚拟主机名的字符串
   PATH_TRANSLATED 当前脚本所在文件系统（非文档根目录）的基本路径
   SCRIPT_NAME 当前脚本的路径 // /dir/demo.php
   REQUEST_URI URI用来指定要访问的页面 // /dir/demo.php?a=aaa&b=bbb
   PHP_AUTH_DIGEST 客户端发送的“Authorization” HTTP头内容
   PHP_AUTH_PW 用户输入的密码
   AUTH_TYPE 认证的类型
   PATH_INFO 包含由客户端提供的、跟在真实脚本名称之后并且在查询语句（query string）之前的路径信息
   ORIG_PATH_INFO 在被PHP处理之前，“PATH_INFO”的原始版本 
```

## 其他

```
    version_compare(str $ver1, str $ver2 [,str $operator])  //比较版本号
        $operator表示操作符，可选：<, lt, <=, le, >, gt, >=, ge, ==, =, eq, !=, <>, ne
        如果省略$operator，返回两个版本号的差值。
    符号@    用于抑制系统运行错误的报告显示
    memory_get_usage    //获取当期内存使用情况
    memory_get_peak_usage   //获取内存使用的峰值
    getrusage   //获取CPU使用情况(Windows不可用)
    uniqid([$prefix])   //获取一个带前缀、基于当前时间微秒数的唯一ID
    highlight_string($str [,$return])   //字符串的语法高亮
        $return：设置为TRUE，高亮后的代码不会被打印输出，而是以字符串的形式返回。高亮成功返回TRUE，否则返回FALSE。
    highlight_file($file [,$return])    //语法高亮一个文件
    __halt_compiler     //中断编译器的执行
    get_browser     //获取浏览器具有的功能
        get_browser ([ string $user_agent [, bool $return_array = false ]] )
        如果设置为 TRUE，该函数会返回一个 array，而不是 object
    eval($code) //把字符串作为PHP代码执行
    gzcompress($str [,$level=-1])   //压缩字符串
    gzuncompress($str)  //解压缩字符串
    gzencode($str [,$level=-1])   //压缩字符串
    gzdecode($str)  //解压缩字符串
    ignore_user_abort($bool) //设置客户端断开连接时是否中断脚本的执行
```

# PHP函数

## 字符串函数

```
    addslashes($str)    //使用反斜线转移字符串
    stripcslashes($str) //反引用一个使用addcslashes转义的字符串
    stripslashes($str)  //反引用一个引用字符串
    chr($ascii) //返回ASCII码的字符
    ord($char)  //返回字符的ASCII码
    substr_count($haystack, $needle)    //计算子串出现的次数
    count_chars($str [,$mode])  统计每个字节值出现的次数
        //0 - 以所有的每个字节值作为键名，出现次数作为值的数组。  
        //1 - 与0相同，但只列出出现次数大于零的字节值。  
        //2 - 与0相同，但只列出出现次数等于零的字节值。  
        //3 - 返回由所有使用了的字节值组成的字符串。  
        //4 - 返回由所有未使用的字节值组成的字符串。 
    crypt($str, [$salt])    //单向字符串散列
    str_split($str [,$len]) //将字符串按长度分割为数组
    explode($separ, $str)   //使用一个字符串分割另一个字符串
    implode([$glue,] $arr)  //将数组元素的值根据$glue连接成字符串
    chunk_split($str [,$len [,$end]])   //将字符串分割成小块
        $len：每段字符串的长度，$end：每段字符串末尾加的字符串(如"\r\n")
    html_entity_decode($str [,$flags [,$encoding]]) //将HTML实体转成字符信息
    htmlentities($str [,$flags [,$encoding]])   //将字符信息转成HTML实体
    htmlspecialchars_decode($str)   //将特殊HTML实体转成字符信息
    htmlspecialchars($str [,$flags [,$encoding]])   //将字符信息转成特殊HTML实体
    lcfirst($str)   //将字符串首字母转成小写
    ucfirst($str)   //将字符串首字母转成大写
    ucwords($str)   //将字符串中每个单词的首字母转换为大写
    strtolower($str)    //将字符串转化为小写
    strtoupper($str)    //将字符串转化为大写
    trim($str [,$charlist]) //去除字符串首尾处的空白字符（或者其他字符）
    ltrim($str [,$charlist])    //去除字符串首段的空白字符（或者其他字符）
    rtrim($str [,$charlist])    //去除字符串末端的空白字符（或者其他字符）
    md5_file($file) //计算指定文件的MD5散列值
    md5($str)   //计算字符串的MD5散列值
    money_format($format, $num) //将数字格式化为货币形式
    number_format($num) //格式化数字
    nl2br($str) //在字符串所有新行之前插入HTML换行标记<br />
    parse_str($str, [$arr]) //解析字符串
    print($str) //输出字符串
    printf      //输出格式化字符串
    sprintf($format [,$args...])    //格式化字符串
    sha1_file   //计算文件的sha1散列值
    sha1        //计算字符串的sha1散列值
    similar_text($first, $second [,$percent])   //计算两个字符串的相似度
        返回在两个字符串中匹配字符的数目，$percent存储相似度百分比
    str_replace($search, $replace, $str [,$count [,$type]])  //子字符串替换
    str_ireplace    //字符串替换(忽略大小写)
    str_pad($str, $len [,$pad [,$type]])  //使用另一个字符串填充字符串为指定长度
        $type：在何处填充。STR_PAD_RIGHT，STR_PAD_LEFT 或 STR_PAD_BOTH
    str_repeat($str, $num)  //重复一个字符串
    str_shuffle($str)   //随机打乱一个字符串
    str_word_count($str [,$format [,$charlist]])    //返回字符串中单词的使用情况
    strcasecmp($str1, $str2)    //二进制安全比较字符串（不区分大小写）
        如果str1小于str2，返回负数；如果str1大于str2，返回正数；二者相等则返回0。
    strcmp($str1, $str2)    //二进制安全字符串比较
    strcoll($str1, $str1)   //基于区域设置的字符串比较(区分大小写，非二进制安全)
    strcspn($str1, $str1 [,$start [,$len]])   //获取不匹配遮罩的起始子字符串的长度
    strip_tags($str)    //从字符串中去除HTML和PHP标记
    strpos($haystack, $needle [,$offset])   //查找字符串首次出现的位置
    stripos($haystack, $needle [,$offset])    //查找字符串首次出现的位置（不区分大小写）
    strripos($haystack, $needle [,$offset])   //计算指定字符串在目标字符串中最后一次出现的位置（不区分大小写）
    strrpos($haystack, $needle [,$offset])   //计算指定字符串在目标字符串中最后一次出现的位置
    strlen($str)    //获取字符串长度
    strpbrk($haystack, $str)    //在字符串中查找一组字符的任何一个字符
    strrev($str)    //反转字符串
        join('', array_reverse(preg_split("//u", $str))); //实现对UTF-8字符串的反转
    strspn$subject, $mask)  //计算字符串中全部字符都存在于指定字符集合中的第一段子串的长度。
    strstr($haystack, $needle)   //查找字符串的首次出现
    stristr($haystack, $needle)   //查找字符串的首次出现(不区分大小写)
    strrchr($haystack, $needle) //查找指定字符在字符串中的最后一次出现
    strtok($str, $token)    //标记分割字符串
    substr_compare($main_str, $str, $offset [,$len) //二进制安全比较字符串（从偏移位置比较指定长度）
    substr_replace$str, $replace, $start [,$len]    //替换字符串的子串
    strtr($str, $from, $to) //转换指定字符
    substr($str, $start [,$len])    //返回字符串的子串
    vfprintf$handle, $format, $args)    //将格式化字符串写入流
    vprintf($format, $args) //输出格式化字符串
    vsprintf($format, $args) //返回格式化字符串
    wordwrap($str [,$width=75 [,$break='\n']])  //打断字符串为指定数量的字串
    
    crc32($str) //计算一个字符串的crc32多项式
        crc32算法[循环冗余校验算法]
        生成str的32位循环冗余校验码多项式。将数据转换成整数。
```

## Math函数

```
    /* Math函数 */
    
    
    base_convert($number, $frombase, $tobase)   //在任意进制之间转换数字
    ceil($float)    //向上取整
    floor($float)   //向下取整
    exp($float) //计算e的指数
    hypot($x, $y)   //计算直角三角形的斜边长
    is_nan($val)    //判断是否为合法数值
    log($arg [,$base=e])  //自然对数
    max($num1, $num2, ...)  //找出最大值
        max($arr)   //找出数组中的最大值
    min($num1, $num2, ...)  //找出最小值
    rand([$min], $max)  //产生一个随机整数
    srand([$seed])  //播下随机数发生器种子
    mt_rand([$min], $max)   //生成更好的随机数
    mt_srand($seed)     //播下一个更好的随机数发生器种子
    pi()    //得到圆周率值
    pow($base, $exp)    //指数表达式
    sqrt($float)    //求平方根
    deg2rad($float) //将角度转换为弧度
    rad2deg($float) //将弧度数转换为相应的角度数
    round($val [,$pre=0]) //对浮点数进行四舍五入
    fmod($x, $y) //返回除法的浮点数余数
```

## PHP运行环境检测函数

```
    php_sapi_name() //返回一个PHP与WEB服务器接口类型的小写字符串
    该函数返回值与常量PHP_SAPI一致！
    接口类型：SAPI(the Server API, SAPI)
    可能值：aolserver、apache、apache2filter、apache2handler、caudium、cgi、cgi-fcgi、cli、 continuity、embed、isapi、litespeed milter、nsapi、phttpd、pi3web、roxen、thttpd、tux、webjames
```

## `fileinfo` 

```
    获取/设置文件信息
    #扩展Fileinfo，配置php.ini
    #extension=php_fileinfo.dll
    finfo_open([$opt]) //创建一个文件信息资源
    finfo_file($finfo, $file [,$opt]) //获取文件信息
    finfo_set_flags($finfo, $opt) //设置文件信息项
    finfo_close($finfo) //关闭文件信息资源
    
    mime_content_type($file) //获取文件的MIME类型
    
    $opt参数选项：
    FILEINFO_MIME_ENCODING 文件编码类型
    FILEINFO_MIME_TYPE 文件MIME类型
    
    
    //目录
    chdir($dir)         改变当前目录
    chroot($dir)        将当前目录改变为当前进程的根目录
    closedir($handle)   关闭目录句柄
    dir($dir)           返回一个目录的实例对象
    getcwd()            取得当前工作目录
    opendir($path)      打开目录句柄
    readdir($handle)    从目录句柄中读取条目
    rewinddir($handle)  倒回目录句柄
    scandir($dir [,$order])     列出指定路径中的文件和目录
    glob($pattern [,$flags])    寻找与模式匹配的文件路径
        $flags：
            GLOB_MARK - 在每个返回的项目中加一个斜线  
            GLOB_NOSORT - 按照文件在目录中出现的原始顺序返回（不排序）  
            GLOB_NOCHECK - 如果没有文件匹配则返回用于搜索的模式  
            GLOB_NOESCAPE - 反斜线不转义元字符  
            GLOB_BRACE - 扩充 {a,b,c} 来匹配 'a'，'b' 或 'c'  
            GLOB_ONLYDIR - 仅返回与模式匹配的目录项 
        查找多种后缀名文件：glob('*.{php,txt}', GLOB_BRACE);
```

## mbstring(多字节字符串)

```
    //需开启mbstring扩展
    mb_strimwidth($str, $start, $width [,$trim [,$encoding]])   //保留指定的子串(并补充)
    mb_stripos($str, $needle [,$offset [,$encoding]])   //查找子串首次出现的位置(忽略大小写)
    mb_strpos($str, $needle [,$offset [,$encoding]])   //查找子串首次出现的位置
    mb_strripos($str, $needle [,$offset [,$encoding]])   //查找子串最后一次出现的位置(忽略大小写)
    mb_strrpos($str, $needle [,$offset [,$encoding]])   //查找子串最后一次出现的位置
    mb_strstr($str, $needle [,$before [,$encoding]])    //返回子串首次出现位置之后(前)的字符串
    mb_stristr($str, $needle [,$before [,$encoding]])    //返回子串首次出现位置之后(前)的字符串(忽略大小写)
    mb_strrchr($str, $needle [,$before [,$encoding]])    //返回字符最后一次出现位置之后(前)的字符串
    mb_strrichr($str, $needle [,$before [,$encoding]])    //返回字符最后一次出现位置之后(前)的字符串(忽略大小写)
    
    mb_strtoupper($str [,$encoding])    //转换成大写
    mb_strtolower($str [,$encoding])    //转换成小写
    
    mb_strlen($str [,$encoding])    //获取字符串长度
    mb_split($pattern, $str [,$limit])  //将字符串分割成数组
    mb_substr($str, $start [,$len [,$encoding]])    //获取字符串的子串
    mb_strcut($str, $start [,$len [,$encoding]])    //获取字符串的子串
    mb_strwidth($str [,$encoding])  //获取字符串的宽度
    mb_substr_count($str, $needle [,$encoding]) //子串在字符串中出现的次数
```

## ob缓存(输出控制)

```
    Output Buffering
    ob_start()  //打开一个输出缓冲区，所有的输出信息不再直接发送到浏览器，而是保存在输出缓冲区里面。
        ob_start('ob_gzhandler'); //将gz编码的数据发送到支持压缩页面的浏览器
    
    ob_clean();            //删除内部缓冲区的内容，不关闭缓冲区(不输出)。
    ob_end_clean();        //删除内部缓冲区的内容，关闭缓冲区(不输出)。
    ob_get_clean();        //返回内部缓冲区的内容，关闭缓冲区。相当于执行ob_get_contents()与ob_end_clean()
    ob_flush();            //发送内部缓冲区的内容到浏览器，删除缓冲区的内容，不关闭缓冲区。
    ob_end_flush();        //发送内部缓冲区的内容到浏览器，删除缓冲区的内容，关闭缓冲区。
    ob_get_flush();        //返回内部缓冲区的内容，并关闭缓冲区，再释放缓冲区的内容。相当于ob_end_flush()并返回缓冲区内容。
    flush();               //将当前为止程序的所有输出发送到用户的浏览器
    
    ob_get_contents();     //返回缓冲区的内容，不输出。
    ob_get_length();       //返回内部缓冲区的长度，如果缓冲区未被激活，该函数返回FALSE。
    ob_get_level();        //Return the nesting level of the output buffering mechanism.
    ob_get_status();       //获取ob状态信息
    
    ob_implicit_flush();   //打开或关闭绝对刷新，默认为关闭，打开后ob_implicit_flush(true)，所谓绝对刷新，即当有输出语句(e.g: echo)被执行时，便把输出直接发送到浏览器，而不再需要调用flush()或等到脚本结束时才输出。
    
    ob_gzhandler               //ob_start回调函数，用gzip压缩缓冲区的内容。
    ob_list_handlers           //List all output handlers in use
    output_add_rewrite_var     //Add URL rewriter values
    output_reset_rewrite_vars  //Reset URL rewriter values
    
    这些函数的行为受php_ini设置的影响：
    output_buffering       //该值为ON时，将在所有脚本中使用输出控制；若该值为一个数字，则代表缓冲区的最大字节限制，当缓存内容达到该上限时将会自动向浏览器输出当前的缓冲区里的内容。
    output_handler         //该选项可将脚本所有的输出，重定向到一个函数。例如，将 output_handler 设置为 mb_output_handler() 时，字符的编码将被修改为指定的编码。设置的任何处理函数，将自动的处理输出缓冲。
    implicit_flush         //作用同ob_implicit_flush，默认为Off。
    
    //ob缓存作用
    1)防止在浏览器有输出之后再使用setcookie()、header()或session_start()等发送头文件的函数造成的错误。其实这样的用法少用为好，养成良好的代码习惯。
    2)捕捉对一些不可获取的函数的输出，比如phpinfo()会输出一大堆的HTML，但是我们无法用一个变量例如$info=phpinfo();来捕捉，这时候ob就管用了。
    3)对输出的内容进行处理，例如进行gzip压缩，例如进行简繁转换，例如进行一些字符串替换。
    4)生成静态文件，其实就是捕捉整页的输出，然后存成文件。经常在生成HTML，或者整页缓存中使用。
```

## `Cookie`

```
    cookie是一种在远程浏览器端储存数据并以此来跟踪和识别用户的机制。
    cookie是HTTP标头的一部分，因此setcookie()函数必须在其它信息被输出到浏览器前调用，这和对header()函数的限制类似。可以使用输出缓冲函数来延迟脚本的输出，直到按需要设置好了所有的cookie或者其它HTTP标头。
    
    // 新增
    setcookie    新增一条cookie信息
    setcookie($name [,$value [,$expire [,$path [,$domain [,$secure [,$httponly]]]]]])
    #注意：setcookie()函数前不能有输出！除非开启ob缓存！
    # 参数说明
    $name    - cookie的识别名称
        使用$_COOKIE['name']抵用名为name的cookie
    $value    - cookie值，可以为数值或字符串，此值保存在客户端，不要用来保存敏感数据
        假定$name参数的值为'name'，则$_COOKIE['name']就可取得该$value值
    $expire    - cookie的生存期限（Unix时间戳，秒数）
        如果$expire参数的值为time()+60*60*24*7则可设定cookie在一周后失效。如果未设定该参数，则会话后立即失效。
    $path    - cookie在服务器端的指定路径。当设定该值时，服务器中只有指定路径下的网页或程序可以存取该cookie。
        如果该参数值为'/'，则cookie在整个domain内有效。
        如果设为'/foo/'，则cookie就在domain下的/foo/目录及其子目录内有效。
        默认值为设定cookie的当前目录及其子目录。
    $domain    - 指定此cookie所属服务器的网址名称，预设是建立此cookie服务器的网址。
        要是cookie能在如abc.com域名下的所有子域都有效，则该参赛应设为'.abc.com'。
    $secure    - 指明cookie是否仅通过安全的HTTPS连接传送中的cookie的安全识别常数，如果设定该值则代表只有在某种情况下才能在客户端与服务端之间传递。
        当设成true时，cookie仅在安全的连接中被设置。默认值为false。
    
    // 读取
    - 浏览器请求时会携带当前域名下的所有cookie信息到服务器。
    - 任何从客户端发送的cookie都会被自动存入$_COOKIE全局数组。
    - 如果希望对一个cookie变量设置多个值，则需在cookie的名称后加[]符号。即以数组形态保存多条数据到同一变量。
        //设置为$_COOKIE['user']['name']，注意user[name]的name没有引号
        setcookie('user[name]', 'shocker');
    - $_COOKIE也可以为索引数组
    
    // 删除
    方法1：将其值设置为空字符串
        setcookie('user[name]', '');
    方法2：将目标cookie设为“已过期”状态。
        //将cookie的生存时间设置为过期，则生存期限与浏览器一样，当浏览器关闭时就会被删除。
        setcookie('usr[name]', '', time()-1);
    
    # 注意：
    1. cookie只能保存字符串数据
    2. $_COOKIE只用于接收cookie数据，不用于设置或管理cookie数据。
        对$_COOKIE进行操作不会影响cookie数据。
        $_COOKIE只会保存浏览器在请求时所携带的cookie数据。
    3. cookie生命周期：
        临时cookie：浏览器关闭时被删除
        持久cookie：$expire参数为时间戳，表示失效时间。
    4. 有效目录
        cookie只在指定的目录有效。默认是当前目录及其子目录。
        子目录的cookie在其父目录或同级目录不可获取。
    5. cookie区分域名
        默认是当前域名及其子域名有效。
    6. js中通过document.cookie获得，类型为字符串
    7. 浏览器对COOKIE总数没有限制，但对每个域名的COOKIE数量和每个COOKIE的大小有限，而且不同浏览器的限制不同。
```

## URL函数

```
    get_headers — 取得服务器响应一个 HTTP 请求所发送的所有标头
    get_meta_tags — 从一个文件中提取所有的 meta 标签 content 属性，返回一个数组
    http_build_query — 生成 URL-encode 之后的请求字符串
    urldecode — 解码已编码的URL字符串
    urlencode — 编码URL字符串
    parse_url — 解析URL，返回其组成部分
        'http://username:password@hostname/path?arg=value#anchor'
        scheme(如http), host, port, user, pass, path, query(在问号?之后), fragment(在散列符号#之后)
```

## `Session`

```
    1. 开启session机制
        session_start()
        注意：session_start()函数前不能有输出！除非开启ob缓存。
    2. 操作数据
        对$_SESSION数组进行操作
    3. 浏览器端保存SessionID，默认为当前域名下的所有目录及其子目录生效。即默认设置cookie的path值为'/'
    4. 服务器保存session数据
        默认保存方式：每个会话都会生成一个session数据文件，文件名为：sess_加SessionID
    5. session可以存储除了资源以外的任何类型数据。
        数据被序列化后再保存到文件中。
    6. $_SESSION的元素下标不能为整型！
        因为只对元素值进行序列化。
        元素内的数组下标无此要求。
    7. 生存周期
        默认是浏览器关闭
            因为浏览器保存的cookie变量SessionID是临时的
            但是服务器端的session数据文件不一定消失（需要等待session的垃圾回收机制来处理）
        可以延长cookie中PHPSESSID变量的生命周期。（不推荐）
        php.ini配置session.gc_maxlifetime
    8. 删除数据
        $_SESSION变量在脚本结束时依然会消失。开启session机制时会造出$_SESSION变量。
        $_SESSION与保存session数据的文件是两个空间。
        unset($_SESSION['key'])只是删除数组内的该元素，不会立即相应到保存session数据的文件上。
            等到脚本结束，才会将$_SESSION的数据写入到该文件中。
        session_destroy()    销毁保存session数据的文件，也不会对该文件写入内容。
            并不删除$_SESSION变量，unset或脚本结束才会删除该变量。
        如何完全删除一个session？需删除3部分
            unset($_SESSION);    
                删除$_SESSION变量后，数据文件并未被改动。如果单独使用unset，则需先置空$_SESSION = array()
            session_destroy();
            setcookie('PHPSESSID', '', time()-1); //保险做法是将其生命周期失效
        整个脚本周期内，只对数据文件读一次、写一次。
    
    // 重写session的存储机制
    # session存储方式
    session.save_handler = user|files|memcache
    # 因数据文件过多导致的问题，可通过分子目录保存进行解决
    PHP配置文件下session.save_path选项，并需手动创建数据存放目录。
    在该配置选项前加层级。分布子目录的原则，利用会话ID的相应字母来分配子目录。仍需手动创建子目录。
    session.save_path = "2; F:/PHPJob/Temp"
    # 多服务器数据共享问题
    # 数据存储操作：
        初始化$open、释放资源$close、读$read、写$write、销毁存储介质$destroy(调用session_destroy时触发该操作)、垃圾回收$gc
    # 会话ID的长度可变。不同的设置方式导致不同长度的会话ID。
    session.hash_function   允许用户指定生成会话ID的散列算法。
        '0' 表示MD5（128 位），'1' 表示SHA-1（160 位）。
    session.hash_bits_per_character    允许用户定义将二进制散列数据转换为可读的格式时每个字符存放多少个比特。
        可能值为 '4'（0-9，a-f），'5'（0-9，a-v），以及 '6'（0-9，a-z，A-Z，"-"，","）。
        总hash长度为128bit，会话ID长度为128/可能值，4->32, 5->26, 6->22
    # 自定义数据存储操作方法
    # 注意：不用关心PHP如何序列化、反序列化、如何得到数据和写入数据，只做与数据存储相关的操作
    session_set_save_handler    设置用户自定义的会话数据存储函数
        bool session_set_save_handler(callable $open, callable $close, callable $read, callable $write, callable $destroy, callable $gc)
    执行顺序：open,  close, read, write, destroy, gc
    # 先设置处理器，再开启会话
    
    # 常用函数
    session_start        开启或恢复会话机制
    session_id            获取或设置当前会话ID
    session_destroy        销毁当前会话的所有数据（销毁数据文件）
    session_name        获取或设置当前会话名称（cookie变量名，默认为PHPSESSID）
    session_save_path    获取或设置当前会话数据文件保存路径
    session_set_save_handler    设置用户自定义的会话数据存储函数
    session_unset        释放所有会话变量(清空$_SESSION数组元素)
    session_encode        将当前会话数据编码为一个字符串
    session_decode        将字符串解译为会话数据
    session_write_close    写入会话数据并关闭会话
    session_register_shutdown    关闭会话
    session_set_cookie_params    设置会话cookie变量，必须在session_start()前使用。
        session_set_cookie_params(0,"/webapp/"); //设置session生存时间
    session_get_cookie_params    获取会话cookie变量。返回包含当前会话cookie信息的数组
    
    # 配置php.ini
    ini_set($varname, $newvalue);
        //该函数的配置只对当前脚本生效
        //并非所有php.ini设置均可用该函数设置
    ini_get($varname)   //获取某配置项信息
    ini_get_all([str $extension])   //返回所有配置项信息的数组
```

## 类型操作函数

```
    //获取/设置类型
    gettype($var) //获取变量的数据类型
    settype($var, $type) //设置变量的数据类型
    
    //类型判断
    is_int
    is_float
    is_null
    is_string
    is_resource
    is_array
    is_bool
    is_object       
    is_numeric      检测变量是否为数字或数字字符串
    
    //转换成指定的数据类型
    boolval
    floatval
    intval
    strval
    
    //强制转换类型
    (int)
    (float)
    (string)
    (bool)
    (array)
    (object)
    (unset) //转换为NULL
    (binary) 转换和 b前缀转换  //转换成二进制
    
    var_dump        打印变量的相关信息。
                    显示关于一个或多个表达式的结构信息，包括表达式的类型与值。
                    数组将递归展开值，通过缩进显示其结构。
    var_export($var [,bool $return]) //输出或返回一个变量的字符串表示
        $return：为true，则返回变量执行后的结果
    print_r         打印关于变量的易于理解的信息
    empty           检查一个变量是否为空
    isset           检测变量是否存在
```

## 数组函数

```
    //统计计算
    count        计算数组中的单元数目或对象中的属性个数
    array_count_values  统计数组中所有的值出现的次数
    array_product       计算数组中所有值的乘积
    array_sum           计算数组中所有值的和
    range        建立一个包含指定范围单元的数组
    
    //获取数组内容
    array_chunk        将一个数组分割成多个
        array array_chunk(array $input, int $size[, bool $preserve_keys]) 
    array_filter    用回调函数过滤数组中的单元
    array_slice     从数组中取出一段
        array array_slice($arr, $offset [,$len [,$preserve_keys]])
    array_keys        返回数组中所有的键名
        array array_keys(array $input[, mixed $search_value[, bool $strict]] )
        如果指定了可选参数 search_value，则只返回该值的键名。否则input数组中的所有键名都会被返回。
    array_values    返回数组中所有的值，并建立数字索引
    
    array_merge        合并一个或多个数组
        一个数组中的值附加在前一个数组的后面。
        如果输入的数组中有相同的字符串键名，则该键名后面的值将覆盖前一个值。
        如果数组包含数字键名，后面的值将不会覆盖原来的值，而是附加到后面。
        如果只给了一个数组并且该数组是数字索引的，则键名会以连续方式重新索引。 
    array_merge_recursive    递归地合并一个或多个数组
    
    //搜索
    in_array            检查数组中是否存在某个值
        bool in_array(mixed $needle, array $haystack[, bool $strict])
    array_key_exists    检查给定的键名或索引是否存在于数组中
        isset()对于数组中为NULL的值不会返回TRUE，而 array_key_exists()会
    array_search        在数组中搜索给定的值，如果成功则返回相应的键名
    
    array_combine    创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值
        如果两个数组的单元数不同或者数组为空时返回FALSE。
    array_rand        从数组中随机取出一个或多个单元，返回键名或键名组成的数组，下标是自然排序的
    array_fill      用给定的值填充数组
        array_fill($start, $num, $value)
    array_flip      交换数组中的键和值
    array_pad       用值将数组填补到指定长度
    array_reverse   返回一个单元顺序相反的数组
    array_unique    移除数组中重复的值
    array_splice    把数组中的一部分去掉并用其它值取代
    
    implode            将数组元素值用某个字符串连接成字符串
    explode($delimiter, $str [,$limit])    //使用一个字符串分割另一个字符串
        $delimiter不能为空字符串""
    
    array_map        将回调函数作用到给定数组的单元上，只能处理元素值，可以处理多个数组
        如果callback参数设为null，则合并多个数组为一个多维数组
    array_walk        对数组中的每个成员应用用户函数，只能处理一个数组，键和值均可处理，与foreach功能相同
        bool array_walk ( array &$array , callback $funcname [, mixed $userdata ] )
    
    //栈：后进先出
    入栈和出栈会重新分配索引下标
    array_push        将一个或多个单元压入数组的末尾（入栈）
    array_pop        将数组最后一个单元弹出（出栈）        使用此函数后会重置(reset())array 指针。
    
    //队列：先进先出
    队列函数会重新分配索引下标
    array_unshift    在数组开头插入一个或多个单元
    array_shift        将数组开头的单元移出数组            使用此函数后会重置(reset())array 指针。
    
    //排序函数
    sort            对数组排序
    rsort            对数组逆向排序
    asort            对数组进行排序并保持索引关系
    arsort            对数组进行逆向排序并保持索引关系
    ksort            对数组按照键名排序
    krsort            对数组按照键名逆向排序
    usort            使用用户自定义的比较函数对数组中的值进行排序
    uksort            使用用户自定义的比较函数对数组中的键名进行排序
    uasort            使用用户自定义的比较函数对数组中的值进行排序并保持索引关联
    natsort            用用“自然排序”算法对数组排序
    natcasesort        用“自然排序”算法对数组进行不区分大小写字母的排序
    array_multisort 对多个数组或多维数组进行排序
    shuffle            将数组打乱
        引用传递参数，返回bool值。
        重新赋予索引键名，删除原有键名
    
    //差集
    array_udiff_assoc   带索引检查计算数组的差集，用回调函数比较数据
    array_udiff_uassoc  带索引检查计算数组的差集，用回调函数比较数据和索引
    array_udiff         用回调函数比较数据来计算数组的差集
    array_diff_assoc    带索引检查计算数组的差集
    array_diff_key      使用键名比较计算数组的差集
    array_diff_uassoc   用用户提供的回调函数做索引检查来计算数组的差集
    array_diff_ukey     用回调函数对键名比较计算数组的差集
    array_diff          计算数组的差集
    //交集
    array_intersect_assoc 带索引检查计算数组的交集
    array_intersect_key 使用键名比较计算数组的交集
    array_intersect_uassoc 带索引检查计算数组的交集，用回调函数比较索引
    array_intersect_ukey 用回调函数比较键名来计算数组的交集
    array_intersect 计算数组的交集
    array_key_exists 用回调函数比较键名来计算数组的交集
    array_uintersect_assoc 带索引检查计算数组的交集，用回调函数比较数据
    array_uintersect 计算数组的交集，用回调函数比较数据
    
    extract($arr [,$type [,$prefix]])   从数组中将变量导入到当前的符号表(接受结合数组$arr作为参数并将键名当作变量名，值作为变量的值)
    compact($var [,...])   建立一个数组，包括变量名和它们的值(变量名成为键名而变量的内容成为该键的值)
```

## 文件、目录

```
    dirname($path)  返回路径中的目录部分
    basename($path [,$suffix])  返回路径中的文件名部分
    pathinfo($path [,$options]) 返回文件路径的信息(数组元素：dirname,basename,extension)
    realpath($path) 返回规范化的绝对路径名
    
    copy($source, $dest)    拷贝文件
    unlink($file)   删除文件
    rename($old, $new)  重命名或移动一个文件或目录
    mkdir($path [,$mode [,$recursive]]) 新建目录
        $mode表示权限，默认0777
        $recursive表示可创建多级目录，默认false
    rmdir($dir)     删除目录(目录必须为空，且具有权限)
    
    file_exists($file)  检查文件或目录是否存在
    is_file($file)      判断文件是否存在且为正常的文件
    is_dir($file)       判断文件名是否存在且为目录
    is_readable($file)  判断文件或目录是否可读
    is_writable($file)  判断文件或目录是否可写
    is_executable($file)    判断给定文件名是否可执行
    is_link($file)      判断给定文件名是否为一个符号连接
    
    tmpfile(void)   建立一个临时文件
    tempnam($dir, $prefix)  在指定目录中建立一个具有唯一文件名的文件
    
    file($file) 把整个文件读入一个数组中
    fopen($filename, $mode [,$use_include_path])
        $mode参数：(加入'b'标记解决移植性)
            'r'     只读方式打开，将文件指针指向文件头。
            'r+'    读写方式打开，将文件指针指向文件头。
            'w'     写入方式打开，将文件指针指向文件头并将文件大小截为零。如果文件不存在则尝试创建之。
            'w+'    读写方式打开，将文件指针指向文件头并将文件大小截为零。如果文件不存在则尝试创建之。
            'a'     写入方式打开，将文件指针指向文件末尾。如果文件不存在则尝试创建之。
            'a+'    读写方式打开，将文件指针指向文件末尾。如果文件不存在则尝试创建之。
            'x'     创建并以写入方式打开，将文件指针指向文件头。
            'x+'    创建并以读写方式打开，将文件指针指向文件头。
    fclose($handle) 关闭一个已打开的文件指针
    fread($handle, $length) 读取文件（可安全用于二进制文件）
    fwrite($handle, $string [,$length]) 写入文件（可安全用于二进制文件）
    rewind($handle) 倒回文件指针的位置
    ftell($handle)  返回文件指针读/写的位置
    fseek($handle, $offset [,$whence])  在文件指针中定位
    feof($handle)   测试文件指针是否到了文件结束的位置
    fgets   从文件指针中读取一行
    fgetss  从文件指针中读取一行并过滤掉HTML标记
    flock($handle, $opt) 轻便的咨询文件锁定
        $opt：LOCK_SH 取得共享锁定（读取的程序）；LOCK_EX 取得独占锁定（写入的程序）；LOCK_UN 释放锁定（无论共享或独占）
    
    
    readfile($file) 读入一个文件并写入到输出缓冲
    fflush($handle) 将缓冲内容输出到文件
    
    touch($file [,$time [,$atime]])   设定文件的访问和修改时间
    fileatime   取得文件的上次访问时间
    filectime   取得文件的inode修改时间
    filegroup   取得文件的组
    fileinode   取得文件的inode
    filemtime   取得文件修改时间
    fileowner   取得文件的所有者
    fileperms   取得文件的权限
    filesize    取得文件大小
    filetype    取得文件类型
```

## 文件上传

```
    enctype="multipart/form-data"   //FORM标签必须的属性
    $_FILES 上传文件信息数组变量
    error   上传错误信息
        0   无错误
        1   文件大小超过php.ini配置
            1) upload_max_filesize 允许上传的最大文件大小
            2) post_max_size 最大的POST数据大小
            3) memory_limit 每个脚本能够使用的最大内存数量(默认128MB)
        2   文件大小超过浏览器表单配置
            MAX_FILE_SIZE   表示表单数据最大文件大小，该元素需在文件上传域之前。(默认2M)
            <input type="hidden" name="MAX_FILE_SIZE" value="102400">
        3   文件只有部分被上传
        4   文件没有被上传
        6,7 临时文件写入时失败
        6   找不到临时文件
        7   文件写入失败
    name    文件名
    type    文件类型
    tmp_name    上传文件临时路径
    size    文件大小
    move_uploaded_file($path, $newpath);    //将上传的文件移动到新位置
    is_uploaded_file($file) //判断是否为POST上传的文件
    //多文件上传
    <input type="file" name="updfile[]" /> //HTML中以数组提交
    $_FILES['updfile']['tmp_name'][0]   //服务器端可访问第一个文件的临时路径，其他属性类似
    
    
    //php.ini配置
    file_uploads = On 是否允许HTTP上传文件
    upload_max_filesize 上传文件大小限制，默认为2M
    post_max_size   post方式表单数据总大小限制，默认为8M
    upload_tmp_dir  上传文件临时目录，默认是系统临时目录
        需设置上传文件临时目录，给其最小权限
    GET方式的最大传输量为2K
```

# 数据库相关

## 数据库操作

```
    #连接认证
    mysql_connect        连接并认证数据库
    #发送SQL语句，接收执行结果
    mysql_query            发送SQL语句
            仅对select, show, explain, describe语句执行成功返回一个资源标识符，其他语句成功返回true。执行失败均返回false。
    #处理结果
    mysql_fetch_assoc    从结果集中取得一行作为关联数组
            每次只取回一条，类似each
        结果集中记录指针
    mysql_fetch_row        从结果集中取得一行作为枚举数组
    mysql_fetch_array    从结果集中取得一行作为关联数组，或数字数组，或二者兼有
        array mysql_fetch_array ( resource $result [, int $ result_type  ] )
        可选参数result_type可选值为：MYSQL_ASSOC，MYSQL_NUM 和 MYSQL_BOTH(默认)
    mysql_free_result    释放结果内存
    #关闭链接
    mysql_close            关闭连接
```

# 代码复用

## 基础代码

- PHP加密解密

```
    <?php
    
    function encryptDecrypt($key, $string, $decrypt)
    {
        if($decrypt){
            $decrypted = rtrim(mcrypt_decrypt(
                MCRYPT_RIJNDAEL_256,
                md5($key),
                base64_decode($string),
                MCRYPT_MODE_CBC,
                md5(md5($key))), "12");
    
            return $decrypted;
        }else{
            $encrypted = base64_encode(mcrypt_encrypt(
                MCRYPT_RIJNDAEL_256,
                md5($key),
                $string,
                MCRYPT_MODE_CBC,
                md5(md5($key))));
    
            return $encrypted;
        }
    }
    
    //以下是将字符串“Hello欢迎您”分别加密和解密
    //加密：
    echo encryptDecrypt('password', 'Hello欢迎您',0).PHP_EOL;
    //解密：
    echo encryptDecrypt('password', 'wA88E1Qs286hmS9tJvkHmiza2lwnW3pDAeiznKVbwEE=',1);
```

- PHP生成随机字符串

```
    function generateRandomString($length = 10) {
        $char = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
        $randomStr = '';
        for ($i = 0; $i < $length; $i++) {
            $randomStr .= $char[rand(0, strlen($char) - 1)];
        }
        return $randomStr;
    }
    
    echo generateRandomString(20);
```

- PHP获取文件扩展名（后缀）

```
    function getSuffix($filename){
      $ext = substr($filename, strrpos($filename, '.'));
    
      return str_replace('.','',$ext);
    }
    
    $filename = '我的文档.doc';
    echo getSuffix($filename);
```

- PHP获取文件大小并格式化

```
    function formatSize($size) {
        $sizes = array(" Bytes", " KB", " MB", " GB", " TB", " PB", " EB", " ZB", " YB");
        if ($size == 0) {
            return('n/a');
        } else {
          return (round($size/pow(1024, ($i = floor(log($size, 1024)))), 2) . $sizes[$i]);
        }
    }
    
    $thefile = filesize('API');
    echo formatSize($thefile);
```

- PHP替换标签字符

```
    function stringParser($string,$replacer){
        $result = str_replace(array_keys($replacer), array_values($replacer),$string);
    
        return $result;
    }
    
    $string = 'The {b}anchor text{/b} is the {b}actual word{/b} or words used {br}to describe the link {br}itself';
    $replace_array = array('{b}' => '<b>','{/b}' => '</b>','{br}' => '<br />');
    
    echo stringParser($string,$replace_array);
```

- PHP列出目录下的文件名

```
    function listDirFiles($DirPath){
        if($dir = opendir($DirPath)){
             while(($file = readdir($dir))!== false){
                    if(!is_dir($DirPath.$file))
                    {
                        echo "filename: $file<br />";
                    }
             }
        }
    }
    
    listDirFiles('../test.com/');
```

- PHP获取当前页面UR

```
    function currentPageURL() {
        $pageURL = 'http';
        if (!empty($_SERVER['HTTPS'])) {
            $pageURL .= "s";
        }
        $pageURL .= "://";
        
        if ($_SERVER["SERVER_PORT"] != "80") {
            $pageURL .= $_SERVER["SERVER_NAME"]
                     .":"
                     .$_SERVER["SERVER_PORT"]
                     .$_SERVER["REQUEST_URI"];
        } else {
            $pageURL .= $_SERVER["SERVER_NAME"]
                     .$_SERVER["REQUEST_URI"];
        }
    
        return $pageURL;
    }
    
    echo currentPageURL();
```

- PHP强制下载文件

```
    function download($filename){
        if ((isset($filename))&&(file_exists($filename))){
           header("Content-length: ".filesize($filename));
           header('Content-Type: application/octet-stream');
           header('Content-Disposition: attachment; filename="' . $filename . '"');
           readfile("$filename");
        } else {
           echo "Looks like file does not exist!";
        }
    }
    
    download('download/test.zip');
```

- PHP获取客户端真实IP

```
    function getIp() {
        if ( getenv("HTTP_CLIENT_IP") &&
            strcasecmp(getenv("HTTP_CLIENT_IP"), "unknown")
        ) {
            $ip = getenv("HTTP_CLIENT_IP");
        } elseif (getenv("HTTP_X_FORWARDED_FOR") &&
                  strcasecmp(getenv("HTTP_X_FORWARDED_FOR"), "unknown")
        ) {
            $ip = getenv("HTTP_X_FORWARDED_FOR");
        } elseif (getenv("REMOTE_ADDR") &&
                  strcasecmp(getenv("REMOTE_ADDR"), "unknown")
        ){
            $ip = getenv("REMOTE_ADDR");
        } elseif (isset ($_SERVER['REMOTE_ADDR']) &&
                  $_SERVER['REMOTE_ADDR'] &&
                  strcasecmp($_SERVER['REMOTE_ADDR'], "unknown")
        ) {
            $ip = $_SERVER['REMOTE_ADDR'];
        } else {
            $ip = "unknown";
        }
    
        return ($ip);
    }
    
    echo getIp();
```

- 获取客户端IP

```
    function getClientIp() {
    
        $ip = NULL;
         if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
    
            $arr = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
            $pos = array_search('unknown',$arr);
            if(false !== $pos) unset($arr[$pos]);
    
            $ip = trim($arr[0]);
         } elseif (isset($_SERVER['HTTP_CLIENT_IP'])) {
           $ip = $_SERVER['HTTP_CLIENT_IP'];
         } elseif (isset($_SERVER['REMOTE_ADDR'])) {
           $ip = $_SERVER['REMOTE_ADDR'];
         }
    
         // IP地址合法验证
         $ip = (false !== ip2long($ip)) ? $ip : '0.0.0.0';
         
         return $ip;
    }
    
    $ip = getClientIp();
    echo $ip;
```

- PHP计算时长(将秒数转化为时分秒格式)

```
    function changeTimeType($seconds) {
        if ($seconds > 3600) {
            $hours   = intval($seconds / 3600);
            $minutes = $seconds % 3600;
            $time    = $hours . ":" . gmstrftime('%M:%S', $minutes);
        } else {
            $time = gmstrftime('%H:%M:%S', $seconds);
        }
        return $time;
    }
    
    $seconds = 3512;
    $seconds = 3712;
    echo changeTimeType($seconds);
```

- 16进制颜色转10进制颜色

```
    function hex2rgb($hexcolor){
        $color = str_replace('#', '', $hexcolor);
    
        if (strlen($color) > 3) {
            $rgb = array(
                'r' => hexdec(substr($color, 0, 2)),
                'g' => hexdec(substr($color, 2, 2)),
                'b' => hexdec(substr($color, 4, 2))
            );
        } else {
            $r = substr($color, 0, 1) . substr($color, 0, 1);
            $g = substr($color, 1, 1) . substr($color, 1, 1);
            $b = substr($color, 2, 1) . substr($color, 2, 1);
            $rgb = array(
                'r' => hexdec($r),
                'g' => hexdec($g),
                'b' => hexdec($b)
            );
        }
    
        return $rgb;
    }
    
    $hex = '#fffeff';
    $hex = '#ffe';
    var_dump(hex2rgb($hex));
```

- Header头文件设置

```
    <?php

    //定义编码
    header( 'Content-Type:text/html;charset=utf-8 ');
    //Atom
    header('Content-type: application/atom+xml');
    //CSS
    header('Content-type: text/css');
    //Javascript
    header('Content-type: text/javascript');
    //JPEG Image
    header('Content-type: image/jpeg');
    //JSON
    header('Content-type: application/json');
    //PDF
    header('Content-type: application/pdf');
    //RSS
    header('Content-Type: application/rss+xml; charset=ISO-8859-1');
    //Text (Plain)
    header('Content-type: text/plain');
    //XML
    header('Content-type: text/xml');
    // ok
    header('HTTP/1.1 200 OK');
    //设置一个404头:
    header('HTTP/1.1 404 Not Found');
    //设置地址被永久的重定向
    header('HTTP/1.1 301 Moved Permanently');
    //转到一个新地址
    header('Location: http://www.jbxue.com/');
    //文件延迟转向:
    header('Refresh: 10; url=http://www.jbxue.com/');
    print 'You will be redirected in 10 seconds';//当然，也可以使用html语法实现// <meta http-equiv="refresh" content="10;http://www.jbxue.com/ />
    // override X-Powered-By: PHP:
    header('X-Powered-By: PHP/4.4.0');
    header('X-Powered-By: Brain/0.6b');
    //文档语言
    header('Content-language: en');
    //告诉浏览器最后一次修改时间
    $time = time() - 60; // or filemtime($fn), etc
    header('Last-Modified: '.gmdate('D, d M Y H:i:s', $time).' GMT');
    //告诉浏览器文档内容没有发生改变
    header('HTTP/1.1 304 Not Modified');
    //设置内容长度
    header('Content-Length: 1234');
    //设置为一个下载类型
    
    header('Content-Type: application/octet-stream');
    
    header('Content-Disposition: attachment; filename="example.zip"');header('Content-Transfer-Encoding: binary');// load the file to send:readfile('example.zip');
    
    // 对当前文档禁用缓存
    
    header('Cache-Control: no-cache, no-store, max-age=0, must-revalidate');
    
    header('Expires: Mon, 26 Jul 1997 05:00:00 GMT'); // Date in the pastheader('Pragma: no-cache');
    
    //设置内容类型:
    
    header('Content-Type: text/html; charset=iso-8859-1');
    
    header('Content-Type: text/html; charset=utf-8');header('Content-Type: text/plain'); //纯文本格式header('Content-Type: image/jpeg'); //JPG***header('Content-Type: application/zip'); // ZIP文件header('Content-Type: application/pdf'); // PDF文件header('Content-Type: audio/mpeg'); // 音频文件header('Content-Type: application/x-shockw**e-flash'); //Flash动画
    
    //显示登陆对话框
    
    header('HTTP/1.1 401 Unauthorized');
    
    header('WWW-Authenticate: Basic realm="Top Secret"');print 'Text that will be displayed if the user hits cancel or ';print 'enters wrong login data';
```

- `Http_request`

```
    public function _request($url, $method, $body=null, $times=1)
        {
            $this->log("Send " . $method . " " . $url . ", body:" . $body . ", times:" . $times);
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, $url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            curl_setopt($ch, CURLOPT_HEADER, true);
            // 设置User-Agent
            curl_setopt($ch, CURLOPT_USERAGENT, self::USER_AGENT);
            // 连接建立最长耗时
            curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, self::CONNECT_TIMEOUT);
            // 请求最长耗时
            curl_setopt($ch, CURLOPT_TIMEOUT, self::READ_TIMEOUT);
            // 设置SSL版本
            curl_setopt($ch, CURLOPT_SSLVERSION, CURL_SSLVERSION_TLSv1);
            curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
            // 如果报证书相关失败,可以考虑取消注释掉该行,强制指定证书版本
            //curl_setopt($ch, CURLOPT_SSL_CIPHER_LIST, 'TLSv1');
            // 设置Basic认证
            curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC);
            curl_setopt($ch, CURLOPT_USERPWD, $this->appKey . ":" . $this->masterSecret);
            // 设置Post参数
            if ($method === self::HTTP_POST) {
                curl_setopt($ch, CURLOPT_POST, true);
            } else if ($method === self::HTTP_DELETE || $method === self::HTTP_PUT) {
                curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
            }
            if (!is_null($body)) {
                curl_setopt($ch, CURLOPT_POSTFIELDS, $body);
            }
    
            // 设置headers
            curl_setopt($ch, CURLOPT_HTTPHEADER, array(
                'Content-Type: application/json',
                'Connection: Keep-Alive'
            ));
    
            // 执行请求
            $output = curl_exec($ch);
            // 解析Response
            $response = array();
            $errorCode = curl_errno($ch);
            $this->log(array($output, $errorCode));
            if ($errorCode) {
                if ($errorCode === 28) {
                    throw new APIConnectionException("Response timeout. Your request has probably be received by JPush Server,please check that whether need to be pushed again.", true);
                } else if ($errorCode === 56) {
                    // resolve error[56 Problem (2) in the Chunked-Encoded data]
                    throw new APIConnectionException("Response timeout, maybe cause by old CURL version. Your request has probably be received by JPush Server, please check that whether need to be pushed again.", true);
                } else if ($times >= $this->retryTimes) {
                    throw new APIConnectionException("Connect timeout. Please retry later. Error:" . $errorCode . " " . curl_error($ch));
                } else {
                    $this->log("Send " . $method . " " . $url . " fail, curl_code:" . $errorCode . ", body:" . $body . ", times:" . $times);
                    $this->_request($url, $method, $body, ++$times);
                }
            } else {
                $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
                $header_size = curl_getinfo($ch, CURLINFO_HEADER_SIZE);
                $header_text = substr($output, 0, $header_size);
                $body = substr($output, $header_size);
                $headers = array();
                foreach (explode("\r\n", $header_text) as $i => $line) {
                    if (!empty($line)) {
                        if ($i === 0) {
                            $headers['http_code'] = $line;
                        } else if (strpos($line, ": ")) {
                            list ($key, $value) = explode(': ', $line);
                            $headers[$key] = $value;
                        }
                    }
                }
                $response['headers'] = $headers;
                $response['body'] = $body;
                $response['http_code'] = $httpCode;
            }
            curl_close($ch);
            return $response;
        }
```

- `MakeDir`

```
    public static function mkdir($dir, $mode = 0777, $makeindex = true)
    {
        if(!is_dir($dir)) {
            if(preg_match('/\.(asp|php|aspx|jsp|cgi)/i', $dir)){
                return false;
            }else if(preg_match('/;/i', $dir)){
                return false;
            }            
            self::mkdir(dirname($dir), $mode, $makeindex);
            @mkdir($dir, $mode);
            if(!empty($makeindex)) {
                @touch($dir.'/index.html'); @chmod($dir.'/index.html', 0777);
            }
        }
        return true;
    }
```

- `createDir`

```
    public static function create($dir, $mode=0777, $makeindex=false)
    {
        $dir = rtrim(str_replace(array('/', '\\'), DIRECTORY_SEPARATOR, $dir), DIRECTORY_SEPARATOR);
        return self::mkdir($dir, $mode, $makeindex);
        //在php设置为base_dir的时候会出错改用self::mkdir递归来创建
        if(!file_exists($dir)){
            if(!$arr = explode(DIRECTORY_SEPARATOR, $dir)){
                return false;
            }
            $path = '';
            foreach ($arr as $k=>$v) {
                $path .= $v . DIRECTORY_SEPARATOR;
                if ($k > 0 && !file_exists($path)) {
                    mkdir($path);
                }
            }
        }
        return true;
    }
```

- `generateQRCodeImage`

```
    // 生成一个二维码图片
    public function generateQRCodeImage(
        $content,
        $errorLevel = 'L',
        $pointSize = 10,
        $margin = 1
    ){
        if (!class_exists('QRcode')) {
            include_once __CORE_DIR.'libs/qrcode/phpqrcode.php';
        }

        QRcode::png($content, false, $errorLevel, $pointSize, $margin);

        exit;
    }
```

- base64转文件输出

```
    function base64ToFile($base64_data, $file){
        if(!$base64_data || !$file){
            return false;
        }
    
        return file_put_contents($file, base64_decode($base64_data), true);
    }
```

- `getJWTString`

```
    public function getJWTString($params = [])
    {
        $header  = base64_encode(json_encode([
            'typ' => 'JWT',
            'alg' => 'SHA256',
        ]));
        $claims = [
            'exp' => __TIME+604800,    // 1 week
            'nbf' => __TIME,
            'iat' => __TIME,
        ];
        $payload = base64_encode(json_encode(array_merge($params, $claims)));
        $signature  = base64_encode(hash_hmac('sha256', $header.'.'.$payload, __CFG::JWT_SECRET_KEY));

        return implode('.', [$header, $payload, $signature]);
    }
```

- `http_get`

```
    private function http_get($url)
    {
        $oCurl = curl_init();
        if (stripos($url, "https://" ) !== FALSE){
            curl_setopt($oCurl, CURLOPT_SSL_VERIFYPEER, FALSE);
            curl_setopt($oCurl, CURLOPT_SSL_VERIFYHOST, FALSE);
        }
        curl_setopt($oCurl, CURLOPT_URL, $url);
        curl_setopt($oCurl, CURLOPT_RETURNTRANSFER, 1);
        $sContent = curl_exec($oCurl);
        $aStatus  = curl_getinfo($oCurl);
        curl_close($oCurl );
        if (intval($aStatus ["http_code"]) == 200) {
            return $sContent;
        } else {
            return false;
        }
    }
```

- `http_post`

```
    private function http_post($url, $param)
    {
        $oCurl = curl_init ();
        if (stripos ( $url, "https://" ) !== FALSE) {
            curl_setopt ( $oCurl, CURLOPT_SSL_VERIFYPEER, FALSE );
            curl_setopt ( $oCurl, CURLOPT_SSL_VERIFYHOST, false );
        }
        if (is_string ( $param )) {
            $strPOST = $param;
        } else {
            $aPOST = array ();
            foreach ( $param as $key => $val ) {
                $aPOST [] = $key . "=" . urlencode ( $val );
            }
            $strPOST = join ( "&", $aPOST );
        }
        curl_setopt ( $oCurl, CURLOPT_URL, $url );
        curl_setopt ( $oCurl, CURLOPT_RETURNTRANSFER, 1 );
        curl_setopt ( $oCurl, CURLOPT_POST, true );
        curl_setopt ( $oCurl, CURLOPT_POSTFIELDS, $strPOST );
        $sContent = curl_exec ( $oCurl );
        $aStatus = curl_getinfo ( $oCurl );
        curl_close ( $oCurl );
        if (intval ( $aStatus ["http_code"] ) == 200) {
            return $sContent;
        } else {
            return false;
        }
    }
```

# 框架相关

## 使用 VarDumper 进行优雅的 PHP 调试

```
    全局安装Symfony VarDumper，这样不仅可以解决样式一次性问题，还可以让你在任何项目中使用Symfony VarDumper，安装方法如下：
    第一步，全局安装：
    composer global require symfony/var-dumper;
    
    
    第二：配置php.ini
    在php.ini中找到auto_prepend_file，然后写上你相对应的路径，比如像下面这样的：
     auto_prepend_file = ${HOME}/.composer/vendor/autoload.php 
    
    
    最后，更新composer
    直接命令行执行：
    composer global update
```

# 服务器

## 网站高并发 大流量访问的处理及解决方法


第一，确认服务器硬件是否足够支持当前的流量。 
普通的P4服务器一般最多能支持每天10万独立IP，如果访问量比这个还要大， 那么必须首先配置一台更高性能的专用服务器才能解决问题 ，否则怎么优化都不可能彻底解决性能问题。 

第二，优化数据库访问。 
前台实现完全的静态化当然最好，可以完全不用访问数据库，不过对于频繁更新的网站， 静态化往往不能满足某些功能。 

缓存技术就是另一个解决方案，就是将动态数据存储到缓存文件中，动态网页直接调用 这些文件，而不必再访问数据库，WordPress和Z-Blog都大量使用这种缓存技术。

如果确实无法避免对数据库的访问，那么可以尝试优化数据库的查询SQL.避免使用 Select * from这样的语句，每次查询只返回自己需要的结果，避免短时间内的大,尽量做到"所查即所得" ,遵循以小表为主,附表为辅,查询条件先索引,先小后大的原则,提高查询效率.
量SQL查询。 

第三，禁止外部的盗链。 
外部网站的图片或者文件盗链往往会带来大量的负载压力，因此应该严格限制外部对于自身的图片或者文件盗链，好在目前可以简单地通过refer来控制盗链，Apache自 己就可以通过配置来禁止盗链，IIS也有一些第三方的ISAPI可以实现同样的功能。当然，伪造refer也可以通过代码来实现盗链，不过目前蓄意伪造refer盗链的还不多， 可以先不去考虑，或者使用非技术手段来解决，比如在图片上增加水印。 

第四，控制大文件的下载。 
大文件的下载会占用很大的流量，并且对于非SCSI硬盘来说，大量文件下载会消耗 CPU，使得网站响应能力下降。因此，尽量不要提供超过2M的大文件下载，如果需要提供，建议将大文件放在另外一台服务器上。 

第五，使用不同主机分流主要流量 
将文件放在不同的主机上，提供不同的镜像供用户下载。比如如果觉得RSS文件占用流量大，那么使用FeedBurner或者FeedSky等服务将RSS输出放在其他主机上，这样别人访问的流量压力就大多集中在FeedBurner的主机上，RSS就不占用太多资源了。 

第六，使用流量分析统计软件。 
在网站上安装一个流量分析统计软件，可以即时知道哪些地方耗费了大量流量，哪些页面需要再进行优化，因此，解决流量问题还需要进行精确的统计分析才可以。我推荐使用的流量分析统计软件是Google Analytics（Google分析）。若还有其他的流量分析软件,欢迎共享交流

# 太多了~~ 看到自己想吐~~