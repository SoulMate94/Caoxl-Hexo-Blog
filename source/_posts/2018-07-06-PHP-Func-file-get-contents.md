---
title: PHP 函数「file_get_contents」
date: 2018-07-06 15:41:38
categories: PHP Func
tags: [PHP, Function]
---

> `file_get_content` - 将整个文件读入一个字符串

<!-- more -->

`file_get_contents()` 函数是用来将文件的内容读入到一个字符串中的**首选方法**。
如果操作系统支持还会使用内存映射技术来增强性能。

# 语法

```
    file_get_contents(path, include_path, context, start, max_length)
```

# 参数

- `path`:  必需. 规定要读取的文件
- `include_path`:  可选. 如果也想在 `include_path` 中搜寻文件的话，可以将该参数设为 `"1"`。
- `context`:  可选. 规定文件句柄的环境.`context` 是一套可以修改流的行为的选项。若使用 `null`，则忽略。
- `start`:  可选. 规定在文件中读取的位置, **该参数是 PHP 5.1 新加的。**
- `max_length`:  可选. 规定读取的字节数. **该参数是 PHP 5.1 新加的。**

# 实例

- 获取并输出网站主页的来源

```
    <?php
    
    $homepage = file_get_contents('http://www.example.com/');
    
    echo $homepage;
```

- 在`include_path`中搜索

```
    <?php
    
    // <= PHP 5
    $file = file_get_contents('./people.txt', true);
    
    // > PHP 5
    $file = file_get_contents('./people.txt', FILE_USE_INCLUDE_PATH);
```

- 读取文件的一部分

```
    <?php
    
    // 从第二十一个字符开始读取14个字符
    $section = file_get_contents('./people.txt', NULL, NULL, 20, 14);
    var_dump($section);
```

- 使用流上下文

```
    <?php
    
    // 创建流
    $opts = array(
        'http'=>array(
          'method'=>"GET",
          'header'=>"Accept-language: en\r\n" .
             "Cookie: foo=bar\r\n"
        )
    );
    
    $context = stream_context_create($opts);
    
    // Open the file using the HTTP headers set above
    $file = file_get_contents('http://www.example.com/', false, $context);
```