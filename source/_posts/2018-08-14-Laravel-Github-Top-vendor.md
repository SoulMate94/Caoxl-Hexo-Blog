---
title: Github Laravel 常用扩展包
date: 2018-08-14 15:25:21
categories: Laravel
tags: [Laravel, Github]
---

> 不要重复造轮子, 是程序员之间的常识

<!-- more -->

# 如何使用扩展包

```
    composer require caoxl/caoxl   # 引入需要的扩展包即可
```

> 以下推荐的扩展包均可在`github`查询具体文档以及使用.

# 1、`laravel/tinker`

> Laravel 框架自带的 REPL 交互控制台，允许你在 Laravel 运行环境中实时调试代码

# 2、`intervention/image`

> 图片处理扩展包，支持裁剪、水印等处理

# 3、`barryvdh/laravel-ide-helper`

> 扩展包能让你的 IDE ( PHPStorm, Sublime ) 实现自动补全、代码智能提示和代码跟踪等功能，大大提高你的开发效率。


# 4、`maatwebsite/excel`

> Excel 处理工具，基于 PhpSpreadsheet ，中文处理时会出现乱码，推荐使用 laravel-snappy 。

# 5、`laravel/socialite`

> 第三方登录相关的类库，对 OAuth 1 & OAuth 2 协议进行了封装，支持自定义驱动，目前开源的 Socialite 驱动已经覆盖了大部分的第三方登录提供商。

# 6、`barryvdh/laravel-cors`

> 跨域资源共享的支持
> 视频文档: https://laravel-china.org/courses/laravel-package/2026/solving-cross-domain-problems-cors-barryvdhlaravel-cors

# 7、`tymon/jwt-auth`

> JWT (JSON Web Token) 用户认证机制，支持 Laravel 和 Lumen

# 8、`barryvdh/laravel-dompdf`

> PDF 操作工具（基于 dompdf ）

# 9、`laravel/passport`

> Laravel 官方维护的 OAuth2 服务扩展包 —— 在 Laravel 中，实现基于传统表单的登陆和授权已经非常简单，但是如何满足 API 场景下的授权需求呢？在 API 场景里通常通过令牌来实现用户授权，而非维护请求之间的 Session 状态。在 Laravel 项目中使用 Passport 可以轻而易举地实现 API 授权认证，Passport 可以在几分钟之内为你的应用程序提供完整的 OAuth2 服务端实现。


# 10、`dingo/api`

> 支持 Laravel 和 Lumen ，功能包括 OAuth 服务、API 版本、Transformor 、节流等。

# 11、`jenssegers/mongodb`

> 让 Laravel 的 Eloquent 模型支持 MongoDB

# 12、`davejamesmiller/laravel-breadcrumbs`

> 可为你的 Laravel 项目快速定制拥有 Bootstrap 风格的面包屑导航。

# 13、`laravel/dusk`

> Laravel Dusk 提供了富有表现力、简单易用的浏览器自动化及测试 API 。默认情况下，Dusk 不需要在你的机器上安装 JDK 或者 Selenium 。而是需要使用单独的 Chrome 驱动 进行安装。当然，你也可以自由使用其他的兼容 Selenium 的驱动程序。


# 14、`laracasts/generators`

> Laracasts 出品的代码快速生成工具

# 15、`spatie/laravel-permission`

> 用户角色权限管理，功能包括模型单独权限、权限缓存优化等。

# 16、`laravel/scout`

> Laravel Scout 是一套基于驱动运行的全文搜索解决方案，目前第三方编写的驱动已经覆盖了主流的专业搜索软件。

# 17、`simplesoftwareio/simple-qrcode`

> PHP 二维码处理工具，基于 BaconQrCode 库。

# 18、`torann/geoip`

> 通过 IP 获取到对应的地理位置信息（GeoIP 数据库）


# 19、`laravel/horizon`

> Horizon 为 Laravel 官方出品的 Redis 队列提供了一个可以通过代码进行配置、并且非常漂亮的仪表盘，并且能够轻松监控队列的任务吞吐量、执行时间以及任务失败情况等关键指标。

# 20、`overtrue/laravel-wechat`

> 微信 SDK for Laravel

# 21、`encore/laravel-admin`

> laravel admin

# 22、三个 HTTP 缓存扩展包

## `barryvdh/laravel-httpcache`

## `spatie/laravel-responsecache`

## `silber/page-cache`

> https://laravel-china.org/topics/10988/three-recommended-http-cache-extensions-for-laravel

# 23、`barryvdh/laravel-debugbar`

> 对 `phpdebugbar` 的封装，用于直观的显示调试及错误信息，提高开发效率，开发必备。

# 24、`itsgoingd/clockwork`

> 配合 Chrome 浏览器下同名插件的调试工具，与 laravel-debugbar 类似，不过侵入性更低。
> https://laravel-china.org/courses/laravel-package/1976/debugging-tool-under-chrome-itsgoingdclockwork

# 参考

> 以上是我自己挑选的几个我认为比较常用的.具体的可以自己查找以下推荐找到适合自己的.

- [GitHub Laravel 扩展包 TOP 250](https://laravel-china.org/projects/filter/laravel-top250?order=downloads_total)