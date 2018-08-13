---
title: Composer 「使用笔记」
date: 2018-02-07 15:46:30
categories: DevTool
tags: [Composer]
---

# Composer 中文镜像 / Packagist 

> 『Composer 中国全量镜像』是由 Laravel China 社区联合 [又拍云](https://www.upyun.com/) 与 [优帆远扬](https://yousails.com/) 共同合作推出的公益项目，旨在为广大 PHP 用户提供稳定和高速的 Composer 国内镜像服务。

<!--more-->

使用 Composer 镜像加速有两种选项：

- 选项一：全局配置，这样所有项目都能惠及（推荐）；

```
    $ composer config -g repo.packagist composer https://packagist.laravel-china.org
```

- 选项二：单独项目配置；

如果仅限当前工程使用镜像，去掉 `-g` 即可，如下：

```
    $ composer config repo.packagist composer https://packagist.laravel-china.org
```

取消镜像

```
    composer config -g --unset repos.packagist
```

## 遇到问题？

`composer `命令后面加上 `-vvv` （是3个v）可以打印出调错信息，命令如下：

```
    $ composer -vvv create-project laravel/laravel blog
    $ composer -vvv require psr/log
```

## FAQ

- 已存在 composer.lock 文件，先删除，再运行 composer install 重新生成。

> 原因：composer.lock 缓存了之前的配置信息，从而导致新的镜像配置无效。

- 使用 `laravel new` 命令创建工程， 这个命令会从 这里 下一个zip包，里面自带了 `composer.lock`，和上面原因一样，也无法使用镜像加速，解决方法：
  
  - 方法一（推荐）：
  不使用 `laravel new`，直接用 `composer create-project laravel/laravel xxx` 新建工程。
  
  - 方法二
  运行 `laravel new xxx`，当看见屏幕出现 `- Installing doctrine/inflector` 时，`Ctrl + C` 终止命令，`cd xxx` 进入，删除 `composer.lock`，再运行 `composer install`。
 
- 缓存多久更新一次？
  - 0时 - 早上7时，这个时间段考虑使用人数不会太频繁，间隔为15分钟
  - 其余时间，间隔为5分钟
  
  > 正常更新速度可在1分内完成 ，但更新太快，会降低CDN命中率，如果总有新文件让CDN去缓存，反而拖慢了速度，所以故意加了些延迟。我们每次采集中还会删减掉数千个僵尸包，以加快传输速度。
  
  
# 安装 Composer

## Linux/Mac：

```
    wget https://dl.laravel-china.org/composer.phar -O /usr/local/bin/composer
    chmod a+x /usr/local/bin/composer
```

## Windows：

1. 直接下载 `composer.phar`，地址：`https://dl.laravel-china.org/composer.phar`
2. 把下载的 `composer.phar` 放到 PHP 安装目录
3. 新建 `composer.bat`, 添加如下内容，并保存

> @php "%~dp0composer.phar" %*

# 查看当前版本

```
    $ composer -V
```

# 升级版本

```
    $ composer selfupdate
```

> **注意** `selfupdate` 升级命令会连接官方服务器，速度很慢。建议直接下载我们的 `composer.phar` 镜像，每天都会更新到最新。