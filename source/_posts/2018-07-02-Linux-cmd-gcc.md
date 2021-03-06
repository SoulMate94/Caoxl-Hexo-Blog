---
title: Linux 命令 「gcc」
date: 2018-07-02 09:14:35
categories: Linux cmd
tags: [Linux]
---

> **gcc命令**使用`GNU`推出的基于`C/C++`的编译器，是开放源代码领域应用最广泛的编译器，具有功能强大，编译代码支持性能优化等特点

<!-- more -->

# 选项

- `-o`:   指定生成的输出文件
- `-E`:   仅执行编译预处理
- `-S`:   将`C`代码转换为汇编代码
- `-wall`:    显示警告信息
- `-c`:   仅执行编译操作, 不进行连接操作

# 参数

- `C`源文件: 指定`C`语言源代码文件

# 实例

## 常用编译命令选项

假设源程序文件名为`test.c`

- 无选项编译链接

```
    gcc test.c
```

将`test.c`预处理、汇编、编译并链接形成可执行文件。这里未指定输出文件，默认输出为`a.out`。

- 选项 `-o`

```
    gcc test.c -o test
```

将`test.c`预处理、汇编、编译并链接形成可执行文件`test`。`-o`选项用来指定输出文件的文件名。

- 选项 `-E`

```
    gcc -E test.c -o test.i
```

将`test.c`预处理输出`test.i`文件。

- 选项 `-S`

```
    gcc -S test.i
```

将预处理输出文件`test.i`汇编成`test.s`文件。

- 选项 `-c`

```
    gcc -c test.s
```

将汇编输出文件`test.s`编译输出`test.o`文件。

- 无选项链接

```
    gcc test.o -o test
```

将编译输出文件`test.o`链接成最终可执行文件`test`。

- 选项 `-O`

```
    gcc -O1 test.c -o test
```

使用编译优化级别1编译程序。级别为`1~3`，级别越大优化效果越好，但编译时间越长

- 多源文件的编译方法

如果有多个源文件，基本上有两种编译方法：

假设有两个源文件为`test.c`和`testfun.c`

1. 多个文件一起编译

```
    gcc testfun.c test.c -o test
```

2. 分别编译各个源文件，之后对编译后输出的目标文件链接。

```
    gcc -c testfun.c #将testfun.c 编译成testfun.o
    gcc -c test.c    #将test.c编译成test.o
    gcc -o testfun.c test.o -o test #将testfun.o和test.o 链接成test    
```

以上两种方法相比较，第一中方法编译时需要所有文件重新编译，而第二种方法可以只重新编译修改的文件，未修改的文件不用重新编译。

