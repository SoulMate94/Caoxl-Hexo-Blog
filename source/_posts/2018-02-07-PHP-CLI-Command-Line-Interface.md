---
title: PHP 命令行模式
date: 2018-02-07 17:06:04
categories: PHP
tags: [PHP, CLI, SAPI]
---

从版本 4.3.0 开始，PHP 提供了一种新类型的 `CLI SAPI`（Server Application Programming Interface，服务端应用编程端口）支持，名为 `CLI`，意为 Command Line Interface，即命令行接口

<!--more-->

# 内置Web Server

PHP 5.4.0起， CLI SAPI 提供了一个内置的Web服务器。

这个内置的Web服务器主要用于本地开发使用，不可用于线上产品环境。

URI请求会被发送到PHP所在的的`工作目录`（Working Directory）进行处理，除非你使用了`-t`参数来自定义不同的目录。

如果请求未指定执行哪个PHP文件，则默认执行目录内的 `index.php` 或者 `index.html` 。如果这两个文件都不存在，服务器会返回 `404` 错误。

## 启动Web服务器

```
    $ cd ~/public_html
    
    $ php -S localhost:8000
```

终端窗口会显示:

```
    PHP 7.1.10 Development Server started at Wed Feb  7 17:12:02 2018
    Listening on http://localhost:8888
    Document root is D:\DeskTop\public_html
    Press Ctrl-C to quit.
```

接着访问http://localhost:8000/和http://localhost:8000/info.php，窗口会显示：

```
    PHP 7.1.10 Development Server started at Wed Feb  7 17:12:02 2018
    Listening on http://localhost:8888
    Document root is D:\DeskTop\public_html
    Press Ctrl-C to quit.
    [Wed Feb  7 17:12:13 2018] ::1:53519 [404]: / - No such file or directory
    [Wed Feb  7 17:12:13 2018] ::1:53520 [404]: /favicon.ico - No such file or directory
    [Wed Feb  7 17:12:18 2018] ::1:53528 [200]: /info.php
    [Wed Feb  7 17:12:42 2018] ::1:53529 Invalid request (Unexpected EOF)
    [Wed Feb  7 17:12:42 2018] ::1:53530 Invalid request (Unexpected EOF)
    [Wed Feb  7 17:12:42 2018] ::1:53531 Invalid request (Unexpected EOF)

```


## 启动时指定根目录

```
    $ cd ~/public_html
    
    $ php -S localhost:8000 -t test/
```

终端窗口显示

```
    PHP 7.1.10 Development Server started at Wed Feb  7 17:14:26 2018
    Listening on http://localhost:8888
    Document root is D:\DeskTop\public_html\test
    Press Ctrl-C to quit.
```

## 使用路由（Router）脚本

请求图片直接显示图片，请求HTML则显示“Welcome to PHP”

```
    <?php
    // router.php
    if (preg_match('/\.(?:png|jpg|jpeg|gif)$/', $_SERVER["REQUEST_URI"]))
        return false;    // 直接返回请求的文件
    else { 
        echo "<p>Welcome to PHP</p>";
    }
    ?>
```

```
    $ php -S localhost:8000 router.php
```

执行之后终端显示：

```
    PHP 7.1.10 Development Server started at Wed Feb  7 17:16:56 2018
    Listening on http://localhost:8000
    Document root is D:\DeskTop\public_html
    Press Ctrl-C to quit.
    [Wed Feb  7 17:17:33 2018] ::1:53688 Invalid request (Unexpected EOF)
    [Wed Feb  7 17:17:33 2018] ::1:53689 Invalid request (Unexpected EOF)
    [Wed Feb  7 17:17:47 2018] ::1:53695 Invalid request (Unexpected EOF)
    [Wed Feb  7 17:18:04 2018] ::1:53713 Invalid request (Unexpected EOF)
    [Wed Feb  7 17:18:04 2018] ::1:53723 [404]: /logo.jpg - No such file or directory
    [Wed Feb  7 17:18:08 2018] ::1:53725 [404]: /logo.png - No such file or directory
    [Wed Feb  7 17:18:17 2018] ::1:53730 [404]: /mylogo.png - No such file or directory
    [Wed Feb  7 17:18:26 2018] ::1:53744 [404]: /mylogo.jpg - No such file or directory
    [Wed Feb  7 17:18:44 2018] ::1:53747 Invalid request (Unexpected EOF)
    [Wed Feb  7 17:18:47 2018] ::1:53754 [200]: /logo.png
```

