---
title: Linux 命令 「awk」
date: 2018-07-05 09:42:30
categories: Linux cmd
tags: [Linux]
---

> `awk`是一种编程语言，用于在`linux/unix`下对文本和数据进行处理。
数据可以来自标准输入(`stdin`)、一个或多个文件，或其它命令的输出。

<!-- more -->

# 语法

```
    awk [options] 'script' var=value file(s)
    awk [options] -f scriptfile var=value file(s)
```

# 命令选项

- `-F fs`:  `fs`指定输入分隔符,`fs`可以是字符串或正则表达式,如`-F`
- `-v var=value`:   赋值一个用户定义变量, 将外部变量传递给`awk`
- `-f scripfile`:   从脚本文件中读取`awk`命令
- `-m [fr] val`:    对val值设置内在限制, 
  - `-mf`选项限制分配给`val`的最大快数目
  - `-mr`选项限制记录的最大数目
> 这两个功能是Bell实验室版awk的扩展功能，在标准awk中不适用


# `awk`模式和操作

## 模式

- /正则表达式/:  使用通配符的扩展集
- 关系表达式:    使用运算符进行操作, 可以是字符串或数字的比较测试
- 模式匹配表达式:  用运算符`~`(匹配) 和 `~!`(不匹配)
- `BEGIN`语句块、`pattern`语句块、`END`语句块

## 操作

操作由一个或多个命令、函数、表达式组成，之间由换行符或分号隔开，并位于大括号内，主要部分是：

- 变量或数组赋值
- 输出命令
- 内置函数
- 控制流语句


# `awk`脚本基本结构

```
    awk 'BEGIN{ print "start" } pattern{ commands } END{ print "end" }' file
```

一个`awk`脚本通常由：**BEGIN语句块**、**能够使用模式匹配的通用语句块**、**END语句块**3部分组成，这三个部分是可选的。
任意一个部分都可以不出现在脚本中，脚本通常是被单引号或双引号中，例如：

```
    awk 'BEGIN{ i=0 } { i++ } END{ print i }' filename
    awk "BEGIN{ i=0 } { i++ } END{ print i }" filename
```

# `awk`的工作原理

```
    awk 'BEGIN{ commands } pattern{ commands } END{ commands }'
```

- 第一步:  执行`BEGIN{ commands }`语句块中的语句
- 第二步:  从文件或标准输入(`stdin`)读取一行，然后执行`pattern{ commands }`语句块，它逐行扫描文件，
从第一行到最后一行重复这个过程，直到文件全部被读取完毕。
- 第三步:  当读至输入流末尾时, 执行`END{ commands }` 语句块

1. **BEGIN语句块**在`awk`开始从输入流中读取行之前被执行，这是一个可选的语句块，比如变量初始化、打印输出表格的表头等语句通常可以写在`BEGIN语句块`中。
2. **pattern语句块**中的通用命令是最重要的部分，它也是可选的。如果没有提供`pattern`语句块，则默认执行{ `print` }，即打印每一个读取到的行，`awk`读取的每一行都会执行该语句块。
3. **END语句块**在`awk`从输入流中读取完所有的行之后即被执行，比如打印所有行的分析结果这类信息汇总都是在END语句块中完成，它也是一个可选语句块。


