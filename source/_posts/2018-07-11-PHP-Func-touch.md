---
title: PHP 函数「touch」
date: 2018-07-11 11:39:01
categories: PHP Func
tags: [PHP, Function]
---

> `touch` - 设定文件的访问和修改时间

<!-- more -->

# 语法

```
    touch(filename, time, atime)
```

# 参数

- `filename` - 必需, 规定要接触的文件
- `time` - 可选, 设置时间,默认是当前系统时间
- `atime` - 可选, 设置访问时间, 默认是当前系统时间

# 说明

> 尝试将由 `filename` 给出的文件的访问和修改时间设定为指定的时间。如果没有设置可选参数 `time`，则使用当前系统时间。
如果给出了第三个参数 `atime`，则指定文件的访问时间会被设为 `atime` 。
>
> 如果成功则返回 `true`，失败则返回 `false`。

**注释: 如果文件不存在, 则会被创建**

# 实例

```php
    <?php
    
    touch('text.txt');
```