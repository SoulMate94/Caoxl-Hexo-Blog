---
title: PHP 函数「strcasecmp」
date: 2018-07-10 17:43:41
categories: PHP Func
tags: [PHP, Function]
---

> `strcasecmp` - 二进制安全比较字符串

<!-- more -->

# 语法

```
    strcasecmp (string1, string2)
```

# 参数

- string1:  必需。规定要比较的第一个字符串。
- string2:  必需。规定要比较的第二个字符串。

# 实例

比较两个字符串 (不区分大小写，HELLO 和 hELLo 输出相同):

```php
    <?php
    
    $var1 = "Hello";
    $var2 = "hello";
    
    if (strcasecmp($var1, $var2) == 0) {
        echo '在不区分大小写的字符串比较中，$var1等于$var2';
    };
```
