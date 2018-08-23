---
title: PHP 系统结构
date: 2018-08-22 11:46:19
categories: PHP
tags: [PHP, 系统结构]
---

> php从下到上是一个4层体系:

<!-- more -->

- **Zend引擎**

​ Zend整体用纯c实现,是php的内核部分,它将php代码翻译(词法、语法解 析等一系列编译过程)为可执行opcode的处理并实现相应的处理方法、实 现了基本的数据结构(如hashtable、oo)、内存分配及管理、提供了相应 的api方法供外部调用,是一切的核心,所有的外围功能均围绕zend实现。

- **Extensions**

围绕着zend引擎,extensions通过组件式的方式提供各种基础服务,我们常 见的各种内置函数(如array系列)、标准库等都是通过extension来实现, 用户也可以根据需要实现自己的extension以达到功能扩展、性能优化等目 的(如贴吧正在使用的php中间层、富文本解析就是extension的典型应 用)。

- **Sapi**

Sapi全称是Server Application Programming Interface,也就是服务端应用编 程接口,sapi通过一系列钩子函数,使得php可以和外围交互数据,这是 php非常优雅和成功的一个设计,通过sapi成功的将php本身和上层应用解 耦隔离,php可以不再考虑如何针对不同应用进行兼容,而应用本身也可以 针对自己的特点实现不同的处理方式。

- **上层应用**

这就是我们平时编写的php程序,通过不同的sapi方式得到各种各样的应用 模式,如通过webserver实现web应用、在命令行下以脚本方式运行等等。

# 构架思想

- 引擎(Zend)+组件(ext)的模式降低内部耦合，中间层(sapi)隔绝web server和php。
- 如果php是一辆车,那么车的框架就是php本身，Zend是车的引擎(发动机) ，Ext下面的各种组件就是车的轮子 Sapi可以看做是公路,车可以跑在不同类型的公路上。而一次php程序的执行就是汽车跑在公路上。 因此,我们需要：**性能优异的引擎+合适的车轮+正确的跑道**。

# Apache和php的关系

Apache对于php的解析,就是通过众多Module中的php Module来完成的。

把php最终集成到Apache系统中,还需要对Apache进行一些必要的设置。

这里,我们就以php的mod_php5 SAPI运行模式为例进行讲解,至于SAPI这个概念后面我们还会详细讲解。

假定我们安装的版本是Apache2 和 Php5,那么需要编辑Apache的主配置文 件http.conf,在其中加入下面的几行内容:

- Unix/Linux环境下:

`LoadModule php5_module modules/mod_php5.so `
`AddType application/x-httpd-php .php`

注:其中modules/mod_php5.so 是X系统环境下mod_php5.so文件的安装位置。

- Windows环境下:

`LoadModule php5_module d:/php/php5apache2.dll`
`AddType application/x-httpd-php .php `

注:其中d:/php/php5apache2.dll 是在Windows环境下php5apache2.dll文件 的安装位置

这两项配置就是告诉Apache Server，以后收到的Url用户请求,凡是以php 作为后缀,就需要调用php5_module模块(mod_php5.so/ php5apache2.dll) 进行处理。

## Apache请求处理循环详解

Apache 请求处理循环的11个阶段都做了哪些事情呢?

### 1、Post-Read-Request阶段

在正常请求处理流程中,这是模块可以插入钩子的第一个阶段。对于那些 想很早进入处理请求的模块来说,这个阶段可以被利用。

### 2、URI Translation阶段

将请求的URL映射到本地文件系统。模块可以在这阶段插入钩子,执行自己的映射逻辑。 mod_alias就是利用这个阶段工作的。

### 3、Header Parsing阶段

检查请求的头部。由 于模块可以在请求处理流程的任何一个点上执行检查请求头部的任务,因 此这个钩子很少被使用。mod_setenvif就是利用这个阶段工作的。

### 4、Access Control阶段

根据配置文件检查是否 允许访问请求的资源。Apache的标准逻辑实现了允许和拒绝指令。 mod_authz_host就是利用这个阶段工作的。

### 5、Authentication阶段

按照配置文件设定的策 略对用户进行认证,并设定用户名区域。模块可以在这阶段插入钩子,实 现一个认证方法。

### 6、Authorization阶段

根据配置文件检查是否 允许认证过的用户执行请求的操作。模块可以在这阶段插入钩子,实现一 个用户权限管理的方法。

### 7、MIME Type Checking阶段

根据请求资源的 MIME类型的相关规则,判定将要使用的内容处理函数。标准模块 mod_negotiation和mod_mime实现了这个钩子。

### 8、FixUp阶段

这是一个通用的阶段,允许模块在内容生成器之前,运行任 何必要的处理流程。和Post_Read_Request类似,这是一个能够捕获任何信 息的钩子,也是最常使用的钩子。

### 9、Response阶段

生成返回客户端的内容,负 责给客户端发送一个恰当的回复。这个阶段是整个处理流程的核心部分。

### 10、Logging阶段

在回复已经发送给客户端之 后记录事务。模块可能修改或者替换Apache的标准日志记录。

### 11、CleanUp阶段

清理本次请求事务处理完成 之后遗留的环境,比如文件、目录的处理或者Socket的关闭等等,这是 Apache一次请求处理的最后一个阶段。

# LAMP 架构

- `Liunx`: 属于操作系统的底层
- `apache`服务器: 属于次服务器,沟通linux和PHP
- `php`: 属于服务端编程语言,通过php_module 模块 和apache关联
- `mysql`和其他web服务: 属于应用服务,通过PHP的Extensions外 挂模 块和mysql关联