---
title: 深入理解「PHP代码」执行过程
date: 2018-01-24 11:31:10
categories: PHP
tags: PHP
---

# 前言

语言是人们进行沟通和交流的表达符号，每种语言都有专属于自己的符号，表达方式和规则。 就编程语言来说，它也是由特定的符号，特定的表达方式和规则组成。语言的作用是沟通，不管是自然语言，还是编程语言，它们的区别在于自然语言是人与人之间沟通的工具， 而编程语言是人与机器之间的沟通渠道。

<!--more-->

就PHP语言来说，它也是一组符合一定规则的约定的指令。 在编程人员将自己的想法以PHP语言实现后，通过PHP的虚拟机（确切的来说应该是PHP的语言引擎Zend）将这些PHP指令转变成C语言 （可以理解为更底层的一种指令集）指令，而C语言又会转变成汇编语言， 最后汇编语言将根据处理器的规则转变成机器码执行。这是一个更高层次抽象的不断具体化，不断细化的过程。

从一种语言到另一种语言的转化称之为`编译`，这两种语言分别可以称之为**源语言**和**目标语言**。 这种编译过程通过发生在目标语言比源语言更低级（或者说更底层）。 语言转化的编译过程是由编译器来完成， 编码器通常被分为一系列的过程：词法分析、语法分析、语义分析、中间代码生成、代码优化、目标代码生成等。 前面几个阶段（词法分析、语法分析和语义分析）的作用是分析源程序，我们可以称之为编译器的前端。 后面的几个阶段（中间代码生成、代码优化和目标代码生成）的作用是构造目标程序，我们可以称之为编译器的后端。 一种语言被称为编译类语言，一般是由于在程序执行之前有一个翻译的过程， 其中关键点是有一个形式上完全不同的等价程序生成。 而`PHP`之所以被称为解释类语言，就是因为并没有这样的一个程序生成， 它生成的是中间代码`Opcode`，这只是`PHP`的一种内部数据结构。 

> 实话实说, 这篇文章是我转载的,只是感觉写的好牛逼~(因为我看不懂),所以自己重新排版了下,以便以后自己能力提升了回来再看看.

# PHP代码的执行的过程

比如我们写一个简单的程序

```
   <?php
       echo "Hello World!";
       $a = 1 + 1;
       echo $a;
   ?> 
```

这个简单的程序他执行过程是怎样的呢？其实，执行过程也正如我们前面所说分为4个步骤。（这里只是指PHP语言引擎Zend执行过程，不包含Web服务器的执行过程。）

```
    1.`Scanning(Lexing)` ,将PHP代码转换为语言片段(Tokens)
    2.`Parsing`, 将Tokens转换成简单而有意义的表达式
    3.`Compilation`, 将表达式编译成Opocdes
    4.`Execution`, 顺次执行Opcodes，每次一条，从而实现PHP脚本的功能。
```

**注:**
  - Opcode是一种PHP脚本编译后的中间语言，就像Java的ByteCode,或者.NET的MSL
  - 现在有的Cache比如APC,可以使得PHP缓存住Opcodes，这样，每次有请求来临的时候，就不需要重复执行前面3步，从而能大幅的提高PHP的执行速度。

## Scanning（Lexing）,将PHP代码转换为语言片段（Tokens）

**那什么是Lexing?** 学过编译原理的同学都应该对编译原理中的词法分析步骤有所了解，`Lex就是一个词法分析的依据表`。

对于PHP在开始使用的是`Flex`，之后改为`re2c`， MySQL的词法分析使用的`Flex`，除此之外还有作为UNIX系统标准词法分析器的Lex等。 这些工具都会读进一个代表词法分析器规则的输入字符串流，然后输出以C语言实做的词法分析器源代码。 这里我们只介绍PHP的现版词法分析器，`re2c`。 在源码目录下的 `Zend/zend_language_scanner.l` 文件是`re2c`的规则文件， 如果需要修改该规则文件需要安装`re2c`才能重新编译，生成新的规则文件。`Zend/zend_language_scanner.c`会根据`Zend/zend_language_scanner.l`,来输入的 PHP代码进行词法分析，从而得到一个一个的“词”。

从PHP4.2开始提供了一个函数叫`token_get_all`,这个函数就可以将一段`PHP`代码 `Scanning`成`Tokens`；

我们用下面的代码使用`token_get_all`函数处理我们开头提到的`PHP`代码。

```
    <?php
    
    echo '<pre/>';
    
    $code = <<< STR
       <?php
           echo "Hello World!";
           $a = 1 + 1;
           echo $a;
       ?>
    STR;
    
         //$tokens = token_get_all($code);
         //print_r($tokens);die;
    
        $tokens = token_get_all($code);
        foreach ($tokens as $key => $token) {
            $tokens[$key][0] = token_name($token[0]);
        }
        print_r($tokens);
    ?>
```

**注：** 为了便于理解和查看，我使用token_name函数将解析器代号修改成了符号名称说明。

*如果有的童鞋想要看原始的，可以将上面代码中的代码注释去掉*

解释器代号列表详见：<u>[解析器代号列表](https://www.php.net/manual/zh/tokens.php)</u>

**得到的结果如下：**

```
    Array
    (
        [0] => Array
            (
                [0] => T_INLINE_HTML
                [1] =>    
                [2] => 1
            )
    
        [1] => Array
            (
                [0] => T_OPEN_TAG
                [1] =>  1
            )
    
        [2] => Array
            (
                [0] => T_WHITESPACE
                [1] =>        
                [2] => 2
            )
    
        [3] => Array
            (
                [0] => T_ECHO
                [1] => echo
                [2] => 2
            )
    
        [4] => Array
            (
                [0] => T_WHITESPACE
                [1] =>  
                [2] => 2
            )
    
        [5] => Array
            (
                [0] => T_CONSTANT_ENCAPSED_STRING
                [1] => "Hello World!"
                [2] => 2
            )
    
        [6] => 
        [7] => Array
            (
                [0] => T_WHITESPACE
                [1] => 
            
                [2] => 2
            )
    
        [8] => 
        [9] => Array
            (
                [0] => T_WHITESPACE
                [1] =>  
                [2] => 3
            )
    
        [10] => Array
            (
                [0] => T_LNUMBER
                [1] => 1
                [2] => 3
            )
    
        [11] => Array
            (
                [0] => T_WHITESPACE
                [1] =>  
                [2] => 3
            )
    
        [12] => 
        [13] => Array
            (
                [0] => T_WHITESPACE
                [1] =>  
                [2] => 3
            )
    
        [14] => Array
            (
                [0] => T_LNUMBER
                [1] => 1
                [2] => 3
            )
    
        [15] => 
        [16] => Array
            (
                [0] => T_WHITESPACE
                [1] => 
           
                [2] => 3
            )
    
        [17] => Array
            (
                [0] => T_ECHO
                [1] => echo
                [2] => 4
            )
    
        [18] => Array
            (
                [0] => T_WHITESPACE
                [1] =>  
                [2] => 4
            )
    
        [19] => 
        [20] => Array
            (
                [0] => T_WHITESPACE
                [1] => 
       
                [2] => 4
            )
    
        [21] => Array
            (
                [0] => T_CLOSE_TAG
                [1] => ?>
                [2] => 5
            )
    
    )
```

分析这个返回结果我们可以发现，源码中的字符串，字符，空格都会原样返回。

每个源代码中的字符，都会出现在相应的顺序处。

而其他的，比如标签，操作符，语句，都会被转换成一个包含三部分的

- 1、**Token ID解释器代号** (也就是在Zend内部的改Token的对应码，比如:`T_ECHO`,`T_STRING`)
- 2、源码中的原来的内容
- 3、该词在源码中是第几行。


## Parsing, 将Tokens转换成简单而有意义的表达式

接下来，就是**Parsing**阶段了，`Parsing`首先会丢弃`Tokens Array`中的多余的空格，

然后将剩余的Tokens转换成一个一个的简单的表达式

```
    echo a constant string
    
    add two numbers together
    
    store the result of the prior expression to a variable
    
    echo a variable
```

`Bison`是一种通用目的的分析器生成器。它将`LALR(1)`上下文无关文法的描述转化成分析该文法的C程序。 使用它可以生成解释器，编译器，协议实现等多种程序。 `Bison`向上兼容`Yacc`，所有书写正确的`Yacc`语法都应该可以不加修改地在`Bison`下工作。 它不但与`Yacc`兼容还具有许多`Yacc`不具备的特性。

`Bison`分析器文件是定义了名为`yyparse`并且实现了某个语法的函数的C代码。 这个函数并不是一个可以完成所有的语法分析任务的C程序。 除此这外我们还必须提供额外的一些函数： 如词法分析器、分析器报告错误时调用的错误报告函数等等。 我们知道一个完整的C程序必须以名为main的函数开头，如果我们要生成一个可执行文件，并且要运行语法解析器， 那么我们就需要有main函数，并且在某个地方直接或间接调用`yyparse`，否则语法分析器永远都不会运行。


在PHP源码中，词法分析器的最终是调用re2c规则定义的lex_scan函数，而提供给Bison的函数则为zendlex。 而yyparse被zendparse代替。


## Compilation, 将表达式编译成Opocdes

之后就是`Compilation`阶段了，它会把`Tokens`编译成一个个`op_array`, 每个`op_arrayd`包含如下5个部分

在PHP实现内部，`opcode`由如下的结构体表如下：

```
    struct _zend_op {
    opcode_handler_t handler; // 执行该opcode时调用的处理函数
    znode result;
    znode op1;
    znode op2;
    ulong extended_value;
    uint lineno;
    zend_uchar opcode; // opcode代码
    };
```

和CPU的指令类似，有一个标示指令的`opcode`字段，以及这个`opcode`所操作的操作数。

PHP不像汇编那么底层， 在脚本实际执行的时候可能还需要其他更多的信息，`extended_value`字段就保存了这类信息。

其中的`result域`则是保存该指令执行完成后的结果。

PHP脚本编译为`opcode`保存在`op_array`中，其内部存储的结构如下：

```
    struct _zend_op_array {
        /* Common elements */
        zend_uchar type;
        char *function_name; // 如果是用户定义的函数则，这里将保存函数的名字
        zend_class_entry *scope;
        zend_uint fn_flags;
        union _zend_function *prototype;
        zend_uint num_args;
        zend_uint required_num_args;
        zend_arg_info *arg_info;
        zend_bool pass_rest_by_reference;
        unsigned char return_reference;
        /* END of common elements */
        zend_bool done_pass_two;
        zend_uint *refcount;
        zend_op *opcodes; // opcode数组
        zend_uint last，size;
        zend_compiled_variable *vars;
        int last_var，size_var;
        // ...
    }
```

如上面的注释，opcodes保存在这里，在执行的时候由下面的execute函数执行：

```
    ZEND_API void execute(zend_op_array *op_array TSRMLS_DC)
    {
        // ... 循环执行op_array中的opcode或者执行其他op_array中的opcode
    }
```

前面提到每条`opcode`都有一个`opcode_handler_t`的函数指针字段，用于执行该`opcode`。

PHP有三种方式来进行`opcode`的处理:`CALL`，`SWITCH`和`GOTO`。

PHP默认使用`CALL`的方式，也就是函数调用的方式， 由于`opcode`执行是每个PHP程序频繁需要进行的操作，

可以使用`SWITCH`或者`GOTO`的方式来分发， **通常GOTO的效率相对会高一些**，

不过效率是否提高依赖于不同的CPU。

在我们上面的例子中，我们的PHP代码会被Parsing成:

```
    * ZEND_ECHO      'Hello World%21'
    * ZEND_ADD       ~0 1 1
    * ZEND_ASSIGN    !0 ~0
    * ZEND_ECHO      !0
    * ZEND_RETURN    1
```

你可能会问了，我们的$a去那里了？这个要介绍`操作数`了，每个操作数都是由以下俩个部分组成：

```
    a)op_type : 为IS_CONST, IS_TMP_VAR, IS_VAR, IS_UNUSED, or IS_CV
      
    b)u,一个联合体，根据op_type的不同，分别用不同的类型保存了这个操作数的值(const)或者左值(var)
```

而对于var来说，每个var也不一样 `IS_TMP_VAR`, 顾名思义，这个是一个临时变量，保存一些`op_array`的结果，以便接下来的`op_array`使用， 这种的操作数的u保存着一个指向变量表的一个句柄（整数），这种操作数一般用~开头。 比如~0,表示变量表的0号未知的临时变量 `IS_VAR` 这种**就是我们一般意义上的变量了**,他们以$开头表示

`IS_CV` 表示 `ZE2.1/PHP5.1` 以后的编译器使用的一种`cache`机制， 这种变量保存着被它引用的变量的地址，当一个变量第一次被引用的时候，就会被CV起来， 以后对这个变量的引用就不需要再次去查找active符号表了，CV变量以！开头表示。

这么看来，我们的`$a`被优化成`!0`了

比如我们使用VLD来查看`opcodes`显示如下：

![Opcodes](https://www.2cto.com/uploadfile/Collfiles/20140404/20140404084908253.jpg)

从上面这个我们看到，是不是和我们之前分析的一样呢。 如上为VLD输出的PHP代码生成的中间代码的信息，说明如下：

- Branch analysis from position 这条信息多在分析数组时使用。
- Return found 是否返回，这个基本上有都有。
- filename 分析的文件名
- function name 函数名，针对每个函数VLD都会生成一段如上的独立的信息，这里显示当前函数的名称
- number of ops 生成的操作数
- compiled vars 编译期间的变量，这些变量是在PHP5后添加的，它是一个缓存优化。 这样的变量在PHP源码中以IS_CV标记。
- op list 生成的中间代码的变量列表 

是不是和前面所说的一样呢。

## Execution,Zend引擎顺次执行Opcodes

最后一步，也就是`Execution，Zend`引擎 顺次执行`Opcodes`，每次一条，从而实现PHP脚本的功能，和机器指令运行相似。 


# 参考

- [深入理解PHP内核](https://www.php-internals.com/)
- [深入理解PHP代码的执行的过程](https://www.2cto.com/kf/201404/290863.html) [这篇文章排版有些问题,估计不是原文]
