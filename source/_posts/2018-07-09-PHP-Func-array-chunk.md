---
title: PHP 函数「array_chunk」
date: 2018-07-09 10:59:07
categories: PHP Func
tags: [PHP, Function]
---

> `array_chunk` - 将一个数组分割成多个数组

<!-- more -->

# 定义和用法

其中每个数组的单元数目由 `size` 参数决定。最后一个数组的单元数目可能会少几个。

可选参数 `preserve_key` 是一个布尔值，它指定新数组的元素是否有和原数组相同的键（用于关联数组），
还是从 0 开始的新数字键（用于索引数组）。默认是分配新的键。

# 语法

```php
    array_chunk(array, size, preserve_key);
```

# 参数

- `array`:    必需. 规定要使用的数组
- `size`:     必需. 整数值,规定每个新数组包含多少个元素
- `preserve_key`: 可选.
  - `true`: 保留原始数组中的键名
  - `false`: 默认. 每个结果数组使用从0开始的新数组索引
  
# 实例

```php
    <?php
    
    $input_array = array('a', 'b', 'c', 'd', 'e');
    
    print_r(array_chunk($input_array, 2));
    print_r(array_chunk($input_array, 2, true));
```