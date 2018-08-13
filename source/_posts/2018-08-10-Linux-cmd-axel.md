---
title: Linux 命令 「axel」
date: 2018-08-10 15:40:14
categories: Linux cmd
tags: [Linux, axel]
---

> `axel`是`Linux`下一个不错的`HTTP/ftp`高速下载工具。支持多线程下载、断点续传，且可以从多个地址或者从一个地址的多个连接来下载同一个文件

<!-- more -->

# 安装

- `CentOS`

```
    yum install axel
```

- `Debian/Ubuntu`

```
    apt-get install axel
```

# 语法

```
    axel 选项 url1 [url2] [url...]
```

# 选项

- `--max-speed=x/-s x`        最高速度x
- `--num-connections=x/-n x`  连接数x
- `--output=f/-o f`           下载为本地文件f
- `--search[=x],-S [x]`       搜索镜像
- `--header=x/-H x`           添加头文件字符串x
- `--user-agent=x/-U x`       设置用户代理
- `--no-proxy/-N`             不使用代理服务器
- `--quiet/-q`                静默模式
- `--verbose/-v`              更多状态信息
- `--alternate/-a`            交替进度指示器
- `--help/-h`                 帮助
- `--version/-V`              版本信息


# 实例

- 如下载`LNMP`安装包指定`10`个线程，存到`/tmp/`:

```
    axel -n 10 -o /tmp/ http://www.linuxde.net/lnmp.tar.gz
```

如果下载过程中下载中断可以再执行下载命令即可恢复上次的下载进度。