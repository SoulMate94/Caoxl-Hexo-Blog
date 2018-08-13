---
title: PHP 函数「文件系统操作」
date: 2018-07-12 11:15:28
categories: PHP Func
tags: [PHP, Function]
---

> PHP文件系统操作常用函数整理

<!-- more -->

# `php.net`

> 太多不想写,自己看吧: <u>[文件系统函数](https://secure.php.net/manual/zh/ref.filesystem.php)</u>


# 文件操作

## 路径相关

- `basename` - 返回路径中的文件名部分

## 文件复制

- `copy` - 拷贝文件

## 文件指针

- `fclose` - 关闭一个已打开的文件指针
- `feof` - 测试文件指针是否到了文件结束的位置
- `fopen` - 打开文件或`URL`
- `fseek` - 在文件指针中定位
- `ftell` - 返回文件指针读/写的位置
- `rewind` - 倒回文件指针的位置

## 文件写入

- `fflush` - 将缓冲内容输出到文件
- `file_put_contents` - 将一根字符串写入文件
- `fputs` - `fwrite`的别名
- `fwrite` - 写入文件 (可安全用于二进制文件)

## 文件读取

- `fgetc` - 从文件指针中读取字符
- `fgets` - 从文件指针中读取一行
- `file_get_contents` - 将整个文件读入一个字符串
- `file` - 将整个文件读入一个数组
- `fpassthru` - 输出文件指针处的所有剩余数据
- `fread` - 读取文件 (可安全用二进制文件)
- `ftruncate` - 将文件截断到给定的长度
- `readfile` - 输出文件

## 文件属性

- `file_exits` - 检查文件或目录是否存在
- `fileatime` - 取得文件的上次访问时间
- `filemtime` - 取得文件修改时间
- `filesize` - 取得文件大小
- `filetype` - 取得文件类型
- `fstat` - 通过已打开的文件指针取得文件信息
- `is_exeutable` - 判断给定文件名是否可执行
- `is_file` - 判断给定文件名是否为一个正常的文件
- `is_readable` - 判断给定文件名是否可读
- `is_uploaded_file` - 判断文件是否是通过`HTTP POST`上传的
- `is_writable` - 判断给定的文件名是否可写
- `is_writeable` - `is_writable`的别名
- `pathinfo` - 返回文件路径的信息
- `realpath` - 返回规范化的绝对路径名
- `rename` - 重命名一个文件或目录
- `stat` - 给出文件的信息

## 新建文件

- `tempnam` - 建立一个具有唯一文件名的文件
- `tmpfile` - 建立一个临时文件
- `touch` - 设定文件的访问和修改时间

## 删除文件

- `unlink` - 删除文件

## `SPL`

### `SplFileInfo`

```php
    <?php
    
    $fileName = 'file.php';
    $fileInfo = new SplFileInfo($fileName);
    
    echo '文件' . $fileName . '的信息如下:' . '</br>';
    echo '文件名:' . $fileInfo->getFilename() . '</br>';
    echo '扩展名:' . $fileInfo->getExtension() . '</br>';
    echo '文件basename' . $fileInfo->getBasename() . '</br>';
    echo '最后访问时间:' . date('Y-m-d H:i', $fileInfo->getATime()) . '</br>';
    echo '最后inode时间:' . date('Y-m-d H:i', $fileInfo->getCTime()) . '</br>';
    echo '最后修改时间:' . date('Y-m-d H:i', $fileInfo->getMTime()) . '</br>';
    echo '文件组:' . $fileInfo->getGroup() . '</br>';
    echo '文件inode:' . $fileInfo->getInode() . '</br>';
    echo '文件拥有者:' . $fileInfo->getOwner() . '</br>';
    echo '文件所在目录:' . $fileInfo->getPath() . '</br>';
    echo '文件所在完整路径:' . $fileInfo->getPathname() . '</br>';
    echo '文件绝对路径:' . $fileInfo->getRealPath() . '</br>';
    echo '文件权限:' . $fileInfo->getPerms() . '</br>';
    echo '文件大小:' . $fileInfo->getSize() . '</br>';
    echo '文件类型:' . $fileInfo->getType() . '</br>';
    echo '是否是目录:' . ($fileInfo->isDir() ? '是' : '否') . '</br>';
    echo '是否是链接:' . ($fileInfo->isFile() ? '是' : '否') . '</br>';
    echo '是否可执行:' . ($fileInfo->isWritable() ? '是' : '否') . '</br>';
    echo '是否可写:' . ($fileInfo->isWritable() ? '是' : '否') . '</br>';
    echo '是否可读:' . ($fileInfo->isReadable() ? '是' : '否') . '</br>';
```

- 在我的`windows`电脑下:

```
    文件file.php的信息如下:
    文件名:file.php
    扩展名:php
    文件basenamefile.php
    最后访问时间:2018-07-12 03:11
    最后inode时间:2018-05-30 10:04
    最后修改时间:2018-07-12 03:11
    文件组:0
    文件inode:0
    文件拥有者:0
    文件所在目录:
    文件所在完整路径:file.php
    文件绝对路径:C:\WWW\Test\file.php
    文件权限:33206
    文件大小:140
    文件类型:file
    是否是目录:否
    是否是链接:是
    是否可执行:是
    是否可写:是
    是否可读:是
```

- 在我的`Mac`下

```
    还没有测试~~~
```

### `SplFileObject`

#### 读取文件

- 1. 方法1

```
    try {
        $fileObject = new SplFileObject($fileName);
        foreach ($fileObject as $line) {
            echo $line . '<br/>';
        }
    } catch (Exception $e) {
        echo $e->getMessage();
    }
```

- 2. 方法2

```
    try {
        $fileObject = new SplFileObject($fileName);
        while ($fileObject->valid()) {
            echo $fileObject->current() . '<br/>';
            $fileObject->next();
        }
    } catch (Exception $e) {
        echo $e->getMessage();
    }
```


#### 写入文件

```
    try {
        $fileObject = new SplFileObject($fileName, 'ab+');
        $fileObject->fwrite('// 写点东西');
    } catch (Exception $e) {
        echo $e->getMessage();
    }
```

## 实例

> 创建一个名为`test`的目录，在目录中创建一个`test.txt`的文件并且项文件中写入"Hello World!"

- 方法1:

```php
    <?php
    
    $rootDir  = '.';
    $newDir   = $rootDir . '/test';
    $filePath = $newDir . '/test.txt';
    $makeDirResult = mkdir($newDir);
    
    if ($makeDirResult) {
        $fileHandler = @fopen($filePath, 'wb+');
        fwrite($fileHandler, 'Hello World!');
        fclose($fileHandler);
        echo "创建目录成功~";
    } else {
        echo "创建目录失败~";
    }
```

- 方法2:

```php
    <?php
    
    $rootDir  = '.';
    $newDir   = $rootDir . '/test';
    $filePath = $newDir . '/test.txt';
    $makeDirResult = mkdir($newDir);
    
    if ($makeDirResult) {
        file_put_contents($filePath, 'Hello World', FILE_APPEND);
        echo "写入成功~";
    }
```

> 返回文件从`X`行到`Y`行的内容

- 方法1

```php
    <?php
    function getContentFromFile($file, $startLine, $endLine) {
        $content = '';
        if (file_exists($file)) {
            $fileHandler = @fopen($file, 'rb');
            $i = 1;
            while (!feof ($fileHandler)) {
                if ($i > $startLine && $i <= $endLine) {
                    $content .= fgets($fileHandler);
                } else {
                    fgets($fileHandler);
                }
                $i++;
            }
            fclose($fileHandler);
        }
    
        return $content;
    }
    
    echo getContentFromFile('./spl.php', 1, 10);
```

- 方法2

```php
    <?php
    
    function getContentFromFile($file, $startLine, $endLine) {
        $content = '';
        $fileObject = new SplFileObject($file);
        $fileObject->seek($startLine);
        $count = $endLine - $startLine;
    
        for ($i = 0; $i <= $count; $i++) {
            $content .= $fileObject->current();
            $fileObject->next();
        }
    
        return $content;
    }
    
    echo getContentFromFile('spl.php', 1, 20);
```

# 目录相关

## 路径相关

- `dirname` - 返回路径中的目录部分

## 目录属性

- `is_dir` - 判断给定文件名是否是一个目录

## 新建目录

- `mkdir` - 新建目录

## 删除目录

- `rmdir` - 删除目录

## 打开目录

- `opendir` - 打开目录句柄
- `readdir` - 从目录句柄中读取条目

## 关闭目录

- `closedir` - 关闭目录句柄

## 扫描目录

- `scandir` - 列出指定路径中的文件和目录

# 参考

- [PHP文件系统操作常用函数整理](https://www.kancloud.cn/backfirecs/php/568353)