---
title: PHP 函数「array_merge」
date: 2018-07-09 10:58:58
categories: PHP Func
tags: [PHP, Function]
---

> `array_merge` - 合并成一个或多个数组

<!-- more -->

# 语法

```
    array_merge(array1, array2, array3...)
```

# 参数

- `array1` 必需, 规定数组
- `array2` 可选, 规定数组
- `array3` 可选, 规定数组

# 实例

```php
    <?php
    
    $array1 = array("color" => "red", 2, 4);
    $array2 = array("a", "b", "color" => "green", "shape" => "trapezoid", 4);
    
    $result = array_merge($array1, $array2);
    
    print_r($result);
```

```php
    Array
    (
        [color] => green
        [0] => 2
        [1] => 4
        [2] => a
        [3] => b
        [shape] => trapezoid
        [4] => 4
    )
```

- 纯数字键名

```php
    <?php
    
    $array1 = array();
    $array2 = array(1 => "data");
    
    $result = array_merge($array1, $array2);
```

别忘了数字键名将会被重新编号！

```php
    Array
    (
        [0] => data
    )
```

- 如果你想完全保留原有数组并只想新的数组附加到后面，用 `+` 运算符:

```php
    <?php
    
    $array1 = array(0 => 'zero_a', 2 => 'two_a', 3 => 'three_a');
    $array2 = array(1 => 'one_b', 3 => 'three_b', 4 => 'four_b');
    
    $result = $array1 + $array2;
    
    var_dump($result);
```