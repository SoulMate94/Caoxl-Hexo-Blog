---
title: Linux 命令 「ssh-keygen」
date: 2018-07-04 14:46:33
categories: Linux cmd
tags: [Linux]
---

> **ssh-keygen命令**用于为`“ssh”`生成、管理和转换认证密钥，它支持`RSA`和`DSA`两种认证密钥。

<!-- more -->

# 语法

```
    ssh-keygen (选项)
```

# 选项

- `-b`:   指定密钥长度
- `-e`:   读取`openssh`的私钥或者公钥文件
- `-C`:   添加注释
- `-f`:   指定用来保存密钥的文件名
- `-i`:   读取未加密的`ssh-v2`兼容的私钥/公钥文件.然后再标准输出设备上显示`openssh`兼容的私钥/公钥
- `-l`:   显示公钥文件的指纹数据
- `-N`:   提供一个新密语
- `-P`:   提供(旧)密语
- `-q`:   静默模式
- `-t`:   指定要创建的密钥类型


