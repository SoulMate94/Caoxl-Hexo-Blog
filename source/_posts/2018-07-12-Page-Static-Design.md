---
title: 页面静态化技术
date: 2018-07-12 09:29:12
categories: PHP
tags: [PHP, 静态化]
---

> 页面静态化的方法，分为两种，一种是伪静态，就是`url`重写，一种是你真的静态化。
下面介绍`PHP`中页面静态化的方法。

<!-- more -->

# 什么是PHP静态化 ?

> `PHP`静态化的简单理解就是使网站生成页面以静态`HTML`的形式展现在访客面前，
`PHP`静态化分纯静态化和伪静态化，两者的区别在于`PHP`生成静态页面的处理机制不同

# PHP生成静态`HTML`页面的方法

## 利用PHP模板生成静态页面

PHP模板实现静态化非常方便，比如安装和使用PHP Smarty实现网站静态化。

## 使用PHP文件读写功能生成静态页面

- `static.php`

```php
    <?php
    
    $out = "<html><head><title>PHP静态化设计</title></head><body>Hello STATIC</body></html>";
    
    $fp = fopen("static.html", "w");
    
    if (!$fp) {
        echo "System Error.";exit();
    } else {
        fwrite($fp, $out);
        fclose($fp);
        
        echo "Success";
    }
```

- `static.html`

```html
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>PHP静态化设计</title>
    </head>
    <body>Hello Caoxl</body>
    </html>
```

## 使用`PHP`输出控制函数(`Output Control`)生成静态页面

输出控制函数(`Output Control`) 也就是使用和控制缓存来生成静态`HTML`页面，也会使用到`PHP`文件读写函数。

```php
    <?php
    
    ob_start();
    
    echo "<html><head><title>PHP静态化设计</title></head><body>Hello OB</body></html>";
    
    $out = ob_get_contents();
    
    ob_end_clean();
    
    $fp = fopen("static.html", "w");
    
    if (!$fp) {
        echo "System Error.";exit();
    } else {
        fwrite($fp, $out);
        fclose($fp);
    
        echo "Success";
    }
```

### `PHP` ob函数介绍

- `ob_start()` - 打开缓冲区
- `ob_get_contents` - 获取缓存中的内容
- `ob_get_clean` - 获取内存中的数据并且清空
- `ob_end_clean` - 清空内存数据

**PHP生成静态页面的思路为**: 
- 首先开启缓存
- 然后输出了HTML内容（你也可以通过include将HTML内容以文件形式包含进来）
- 之后获取缓存中的内容，清空缓存后通过PHP文件读写函数将缓存内容写入到静态HTML页面文件中


