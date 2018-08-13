---
title: PHP 函数「array_combine」
date: 2018-07-24 17:21:56
categories: PHP Func
tags: [Function]
---

> 通过合并两个数组来创建一个新数组，其中的一个数组元素为键名，另一个数组元素为键值

<!-- more -->

# 语法

```
    array_combine($keys, $values);
```

# 参数

- `keys` - 必需, 键名数组
- `values` - 必需, 键值数组

# 实例

```
    $name = array('caoxl', 'xian', 'liang');
    $age  = array('18', '20', '23');
    
    $arr  = array_combine($name, $age);
    
    var_dump($arr);
```
