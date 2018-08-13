---
title: PHP 函数「file_put_contents」
date: 2018-07-06 15:41:45
categories: PHP Func
tags: [PHP, Function]
---

> `file_put_contents` — 将一个字符串写入文件

<!-- more -->

和依次调用 `fopen()`, `fwrite()` 以及 `fclose()` 功能一样。

# 语法

```
    file_put_contents(file, data, mode, context)
```

# 参数

- `file`:  必需. 规定要写入数据的文件, 文件文件不存在, 则创建一个新文件
- `data`:  可选. 规定要写入文件的数据. 可以是字符串、数组或数据流
- `mode`:  可选. 规定如何打开/写入文件. 可能的值
  - `FILE_USE_INCLUDE_PATH`: 在 `include` 目录里搜索 `filename`
  - `FILE_APPEND`: 如果文件 `filename` 已经存在，追加数据而不是覆盖。
  - `LOCK_EX`: 在写入时获得一个独占锁。
- `context`: 可选. 规定文件句柄的环境. `context` 是一套可以修改流的行为的选项。若使用 null，则忽略。

# 实例

- 简单的用法示例

```
    <?php
    
    $file = 'people.txt';
    
    $current = file_get_contents($file);
    
    $current .= "John Smith\n";
    
    file_put_contents($file, $current);
```

- 使用标志

```
    <?php
    
    $file = 'people.txt';
    
    $person = "John Smith\n";
    
    file_put_contents($file, $person, FILE_APPEND | LOCK_EX);
```