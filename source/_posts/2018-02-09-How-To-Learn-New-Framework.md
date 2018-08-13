---
title: 如何上手新框架「基础篇」
date: 2018-02-09 17:17:47
categories: Framework
tags: [PHP, Framework, Laravel, Lumen, ThinkPHP]
---

不知不觉, 2018都已经到了2月份了,放假前总结一波

>  不同的框架就是PHP不同的衣服, 外表不同, 内在一样

<!--more-->

# 常用文档

## 框架

> - Laravel <u>[中文](https://d.laravel-china.org/docs/5.5)</u>， <u>[英文](https://laravel.com/docs/5.6)</u>

> - Lumen <u>[中文](https://lumen.laravel-china.org/docs/5.3)</u>， <u>[英文](https://lumen.laravel-china.org/docs/en/5.3)</u>

> - Laravel-admin <u>[中文](http://laravel-admin.org/docs/#/zh/)</u>, <u>[英文](http://laravel-admin.org/docs/#/)</u>
 
> - ThinkPHP 5 <u>[中文](https://www.kancloud.cn/manual/thinkphp5_1/353946)</u>, <u>[]()</u>

## API

> - Laravel API <u>[英文](https://laravel.com/api/5.6/index.html)</u>
> - Dingo API <u>[中文](https://github.com/liyu001989/dingo-api-wiki-zh)</u>，<u>[英文](https://github.com/dingo/api/wiki)</u>

# **要做什么? 怎么做? 为什么?**

# What ?

> 对于一个新框架, 我们要学习什么?

对于一个框架,我们需要知道:

**必知**

- 项目入口

- 配置文件 (config/ENV)

- 环境要求 (PHP/MySQL版本等)

- MVC (Controller, Model, View)

- Route (路由)

- ORM/Eloquent (对象关系映射)

**扩展**

- 访问权限 (OAuth, JWT)

- 缓存 (Redis/Memcache)

- Session/Cookie

- Composer

- 任务调度/定时任务 (Schedule/Crontab)


**第三方**

- 第三方登录 (微信/QQ/微博/云联惠 等)

- 第三方支付 (微信/支付宝/快钱/网银 等)

- 短信服务 (阿里云通信/极光推送/腾讯信鸽/接口网 等)

- 地图服务 (百度地图/高德地图 等)

- 图片存储 (七牛云对象存储 等)

- 微信公众号

**深入**

- 网站优化 (CDN加速, SEO优化)

- 服务器部署 (Linux/Nginx/CentOS/Ubantu 等)

- 自动部署 (WebHook/Github/Gitlab)

- 网站加密 (HTTPS/SSL)

- Git 团队工作流

- 团队开发规范/测试流程规范/前后端分离规范

> 不同的框架 有不同的操作 当流程大体一致


# How ?

## `Laravel`

> - <u>[Laravel 使用笔记](http://blog.caoxl.com/2018/01/16/Laravel-Dev-Notes/)</u>

> - <u>[深入理解 Laravel](http://blog.caoxl.com/2018/01/16/Dig-Into-Laravel/)</u>

> - <u>[Laravel5.5 with DA - LA](http://blog.caoxl.com/2018/01/31/Laravel-5-5-With-Dingo-API-And-LaravelAdmin/)</u>

### 项目入口

`public/index.php`

注: 查看 `public/index.php` 和 `bootstrap/app.php`两个文件

### 配置文件 (`config` / `ENV`)

`.env`和`config/ 目录下的各个文件`

注: `.env` 加入 .gitignore中, 项目中使用`.env.example`方便团队开发

### 环境要求 (PHP/MySQL版本等)

根据不同项目要求,选择不同的版本

> 如我工作中: 
> - 二次开发江湖外卖 需要 PHP 5.4-nts (nts 非线程安全)
> - Lumen框架/Laravel-admin框架中 需要 PHP 7.1+ MySQL 5.7

### MVC (Controller, Model, View)

- [Controller](https://d.laravel-china.org/docs/5.5/controllers)
  - [Middleware](https://d.laravel-china.org/docs/5.5/middleware)
  - [Request](https://d.laravel-china.org/docs/5.5/requests)
  - [Response](https://d.laravel-china.org/docs/5.5/responses)

- [Model](https://d.laravel-china.org/docs/5.1/database)
  - [DB](https://cs.laravel-china.org/#db)
  - [Migrations](https://d.laravel-china.org/docs/5.5/migrations)
  - [Seeders](https://d.laravel-china.org/docs/5.5/seeding#introduction)
  - [Redis](https://d.laravel-china.org/docs/5.5/redis)
  
- [View](https://d.laravel-china.org/docs/5.5/views)

### Route (路由)

- [routes](https://d.laravel-china.org/docs/5.5/routing)

```
    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);
```

### ORM/Eloquent (对象关系映射)

- [Eloquent](https://d.laravel-china.org/docs/5.5/eloquent)
  - [Collection](https://d.laravel-china.org/docs/5.5/eloquent-collections)
  - [Resource](https://d.laravel-china.org/docs/5.5/eloquent-resources)
  - [Serialization](https://d.laravel-china.org/docs/5.5/eloquent-serialization)
  - [Relationship](https://d.laravel-china.org/docs/5.5/eloquent-relationships)

### 核心架构

- [Facades](https://d.laravel-china.org/docs/5.5/facades)

- [Contracts](https://d.laravel-china.org/docs/5.5/contracts)

- [Container](https://d.laravel-china.org/docs/5.5/container)

- [Providers](https://d.laravel-china.org/docs/5.5/providers)

### 扩展

- [Passport](https://d.laravel-china.org/docs/5.5/passport)

## `Lumen`

> 文档: https://lumen.laravel-china.org/docs/5.3

> - <u>[Lumen5.4-开发记录](http://blog.caoxl.com/2018/01/05/Lumen54-Dev-Log-Reprint/)</u>

> - <u>[JWT-Lumen5.4-自定义](http://blog.caoxl.com/2018/01/17/JWT-Lumen54-Custom/)</u>

** `Lumen` 是简洁版的 `Laravel` **

学习路线基本和官方文档一致,以下挑选几个开发必知.

### 基本功能

- [路由](https://lumen.laravel-china.org/docs/5.3/routing)
- [中间件](https://lumen.laravel-china.org/docs/5.3/middleware)
- [控制器](https://lumen.laravel-china.org/docs/5.3/controllers)
- [请求](https://lumen.laravel-china.org/docs/5.3/requests)
- [响应](https://lumen.laravel-china.org/docs/5.3/responses)

### 更多功能

- [用户认证](https://lumen.laravel-china.org/docs/5.3/authentication)
- [用户授权](https://lumen.laravel-china.org/docs/5.3/authorization)
- [数据库](https://lumen.laravel-china.org/docs/5.3/database)
- [服务容器](https://lumen.laravel-china.org/docs/5.3/container)
- [服务提供者](https://lumen.laravel-china.org/docs/5.3/providers)
- [数据验证](https://lumen.laravel-china.org/docs/5.3/validation)
- [错误与日志](https://lumen.laravel-china.org/docs/5.3/errors)
- [缓存](https://lumen.laravel-china.org/docs/5.3/cache)
- [事件](https://lumen.laravel-china.org/docs/5.3/events)
- [队列](https://lumen.laravel-china.org/docs/5.3/queues)

## `Laravel-admin`

> http://laravel-admin.org/docs/#/zh/

> - <u>[Laravel-admin-开发记录](http://blog.caoxl.com/2018/01/04/Laravel-admin-Dev-Notes/)</u>

> - <u>[Laravel5.5 with DA - LA](http://blog.caoxl.com/2018/01/31/Laravel-5-5-With-Dingo-API-And-LaravelAdmin/)</u>

**Laravel-admin 是一个基于Laravel的后台框架,用于开发后台很高效**

### 基础使用

- [模型表单 Grid](http://laravel-admin.org/docs/#/zh/model-grid)
- [模型表单 Form](http://laravel-admin.org/docs/#/zh/model-form)
- [帮助工具](http://laravel-admin.org/docs/#/zh/extension-helpers)
- [自定义登陆](http://laravel-admin.org/docs/#/zh/custom-authentication)

## 二次开发

> - <u>[二次开发之江湖外卖](http://blog.caoxl.com/2018/01/05/JHWM-Second-Dev-Log-Reprint/)</u>

**二次开发主要问题是有很多坑, 并且没有文档告诉你如何解决.你需要自己去看代码依葫芦画瓢去解决**


## ThinkPHP 3.2/5.0

> 待更新,需要总结以前的工作,以及以前的笔记.2018一年主要是在前三个框架中度过的


# Why ?

每个框架都有**各自的优点长处**, 也因为不同**公司的性质不一样**, 要做的**项目要求不一样**.


# 总结

对于**工作**:你主要是需要知道 **How ?** ,因为很多时候你需要考虑的是做出功能,好的程序员不仅仅是做出功能,而是做好功能

对于**学习**以及对**编程**:你需要知道 **Why ?**, 甚至更深层次的, 只有知道工具是怎么造的, 才能自己造出更好的工具

> 未完待续~~ 不定时更新