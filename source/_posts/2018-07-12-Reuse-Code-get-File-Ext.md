---
title: 「代码复用」获取文件后缀
date: 2018-07-12 11:46:09
categories: 代码复用
tags: [PHP, 代码复用]
---

> 很有必要开启一个代码复用的专栏, 作为程序员切记重复造轮子

<!-- more -->

# 直接上源码


```
    <?php
    
    // 获取扩展名
    function file_ext($file){
        return strtolower(pathinfo($file, 4));
        // return strtolower(pathinfo($file,PATHINFO_EXTENSION));
    }
    
    echo file_ext('./php.php');
    // print_r(pathinfo('./php.php'));
    // Array ( [dirname] => . [basename] => php.php [extension] => php [filename] => php )
```

- `strtolower` - 将字符串转化为小写
- `pathinfo` - 返回文件路径的信息

# `pathinfo`

## 语法

```
    pathinfo(path, options)
```

## 参数

- `path` - 必需. 规定要检查的路径
- `options` - 可选. 规定要返回额数组元素.默认是all. 可能的值:
  - `PATHINFO_DIRNAME` - 只返回 `dirname`
  - `PATHINFO_BASENAME` - 只返回 `basename`
  - `PATHINFO_EXTENSION` - 只返回 `extension`
