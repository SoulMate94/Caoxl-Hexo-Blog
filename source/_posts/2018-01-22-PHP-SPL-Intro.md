---
title: PHP SPL 简介
date: 2018-01-22 17:03:29
categories: PHP
tags: [PHP, SPL, 标准库]
---

每种语言应该都有自己的标准库，而 **SPL 是 PHP 的标准库**，一般编码中用得比较少，但是其提供的基本数据结构实现在解决相对复杂的应用场景还是很重要的。

# What ?

> 用于解决典型，常见问题的一组接口与类的集合。

所谓的常见问题指的有：

- **数据结构（数学建模）：** 解决数据怎么存储的问题
- **元素遍历：** 数据怎么查看的问题。
- **常用方法的统一调用：** 通用方法（数组或集合的大小）、自定义遍历等。
- **类定义在自动装载：** 让 php 适合大型项目的管理需求，把功能的实现分散到不同的文件中。

> 类定义的自动装载解决的是项目程序在运行的时候自动判断什么时候需要什么类文件。

<!--more-->

## Basic Framework

- 数据结构
- 基础接口
- 基础函数
- 迭代器
- 异常
- 其他


# SPL 基础函数

## autoload

为了初始化 PHP 中的类对象，需要通过一定的方法寻找到类的定义。通常情况下，类会定一个在一个单独的文件中。

> Yii 的延迟加载就用到了这个特性。

### 类载入的基本流程

- 先在**当前文件**中查找，如果找到，则初始化类，然后结束。

- 如果没找到，就依次调用 `spl_autoload_register()` 注册的方法去找，如果找到，就初始化类，然后结束。

- 如果没找到，就调用文件的 `__autoload()` 函数去找，如果找到就初始化类，然后结束。

- 如果还是没找到，就会**抛出一个类找不到的异常**。


## 迭代器

- `iterator_apply：`为迭代器中的每个元素调用一个用户自定义函数。
- `iterator_count：`计算迭代器中元素的个数。
- `iterator_to_array：`将迭代器中的元素拷贝到数组。

## 类信息

- `class_implements：`返回指定类实现的所有接口。

> 可以用 `instanceof` 判断某个对象是够实现了某个接口活着是某个类的实例。

- `class_parents：`返回指定类的父类，如果继承了多次，则会把所有的父类都打印出来。


# SPL 基础接口

## Countable

继承该接口的类可以直接调用 `count()` 得到元素个数。

## OuterIterator

如果想迭代器进行一定的处理之后再返回，可以用这个接口。

`IteratorIterator` 是 `OuterIterator` 的实现，扩展的时候，可以直接继承 `IteratorIterator`，就可以实现最基本的操作 了。

## RecursiveIterator

可以对多层结构的迭代器进行迭代，比如遍历一棵树。

所有具有文件层次结构特点的数据都可以用这个接口遍历，比如文件夹。

### 关键方法

- `hasChildren：`用于判断当前节点是否存在子节点。
- `getChildren：`用于得到当前节点子节点的迭代器。

### SPL 中实现该接口的类

- RecursiveArrayIterator
- RecursiveCachingIterator

**Recursive 开头的类都能够进行多层次结构化的遍历。**


## SeekableIterator

可以通过 `seek` 方法定位到集合里面的某个特定元素。**seek 方法的参数就是元素的位置，从 0 开始算**。

### 实现该接口的类

- ArrayIterator
- DirectoryIterator
- FilesystemIterator：遍历文件系统。
- GlobIterator
- RecursiveArrayIterator
- RecursiveDirectoryIterator


# SAP 迭代器

## What is Iterator ?

怎样获得链表中每个节点的信息？

通过某种统一的方式遍历链表活着数组中的元素的过程，叫做**迭代遍历**，而这种统一的遍历工具，就叫做**迭代器**。

遍历数组的时候，如果使用 `foreach`，其背后也是通过迭代器对象去执行遍历的。

## Why using Iterator ?

使用迭代器的目的都是为了**遍历**。


## 常用迭代器

### ArrayIterator

- 用于遍历数组
- 用于跳过某些元素（seek）
- 用于排序


### AppendIterator

按顺序迭代访问几个不同的迭代器，例如，希望在一次循环中迭代访问两个或更多的组合。

### MultipleIterator

用于把多个 `Iterator` 里面的数据组合成为一个整体来访问。

# SPL 常用数据结构

数据结构是与语言无关的，因为它解决的是数据在计算机中是如何存储、组织以及展示的问题。**学习 `SPL` 常用数据结构的目的就是会用 `SPL` 里面提供的方法去间接操作数据结构。**

SPL 提供的数据结构有：

- 双向链表
- 堆栈
- 队列
- 堆
- 降序堆
- 升序堆
- 优先级队列
- 定长数组
- 对象容器

> 其实并不需要亲自实现，程序员作为编程语言的使用者只是调用这些实现而已。（大致所有高级语言皆如此）

## 双向链表

- **bottom：** 最先添加到链表中的节点叫做 Bottom，也称为头部。
- **top：** 最后添加到链表中的节点叫做 Top，也称为尾部
- **链表指针：** 是一个当前关注节点的标识，可以指向任意节点。
- **当前节点：** 链表指针指向的节点称为当前节点。

### 节点

- **节点名称：** 可以在链表中唯一标识一个节点的名称，我们通常又称为节点的 `Key` 或 `Offset`。
- **节点数据：** 存放在链表中的应用数据，即 Value。


### SplDoublyLinkedList 类

- 操作当前节点：
  - rewind：使链表的当前指针指向链表的底部（头部）。
  - current：指向链表当前节点的指针，必须在调用之前先调用 rewind，当前节点被删除后，会指向一个空节点。
  > 使用时注意判断 current 的值是否指向一个空节点。
  - next：使链表当前节点的指针指向下一个节点，current 的返回值会改变。
  - prev
  
- 增加节点：
  - push：向链表的顶部（尾部）插入一个节点。
  - unshift：向链表的底部插入一个节点。
  
- 删除节点
  - pop：获取链表中的顶部节点，并且从链表中删除这个节点；此操作不会改变当前指针的位置。
  - shift：删除一个链表底部节点。
  
- 定位操作
  - bottom：获得链表底部元素，当前指针位置不变。
  - top：获得链表顶部元素，当前指针位置不变。
  
- 特定节点操作
  - offsetExists
  - offsetGet
  - offsetSet
  - offsetUnset

## 堆栈

堆栈和队列刚好相反，具有后进先出（FILO）特点的连续型数据结构。

### SplStack

继承自 `SplDoublyLinkedList`，操作如下：

- **push：** 压入堆栈。
- **pop：** 弹出堆栈。

### SplQueue

继承自 `SplDoublyLinkedList`，操作如下：

- **enqueue：** 进入队列
- **dequeue：** 退出队列


# 其他 SPL 类库

- 文件处理类库
  - `SplFileInfo：`用于获得文件的基本信息，比如修改时间、大小、目录等信息。
  - `SplFileObject：`用于操作文件的内容。

- `SplFixedArray：` 定长数组。

- `SplMinHeap：` 堆栈。（insert、extract）

# 参考

- [PHP标准库 (SPL)](http://php.net/manual/zh/book.spl.php)
- [PHP SPL笔记](http://www.ruanyifeng.com/blog/2008/07/php_spl_notes.html)
- [Github 示例代码](https://github.com/ckwongloy/lbd/tree/master/spl)