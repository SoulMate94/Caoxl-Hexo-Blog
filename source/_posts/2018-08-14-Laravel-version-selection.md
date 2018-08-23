---
title: Laravel 版本选择
date: 2018-08-14 16:02:27
categories: Laravel
tags: [Laravel]
---

> 原文地址: https://laravel-china.org/docs/laravel-specification/5.5/laravel-version-selection/512
> > 转载到自己博客是为了加深记忆和方便查询

<!-- more -->

以下是 Laravel 的版本路线图：

|版本|发布日期|版本类型|维护周期|
|:----|:-----|:-----|:----|
|Laravel 5.1|2015 年 6 月|LTS 长久支持|Bug 修复 2017 年 6 月份，安全修复 2018 年 6 月份|
|Laravel 5.2|2015 年 12 月|一般发行|提供 6 个月的 Bug 修复支持，一年的安全修复支持|
|Laravel 5.3|2016 年 8 月|一般发行|提供 6 个月的 Bug 修复支持，一年的安全修复支持|
|Laravel 5.4|2017 年 1 月|一般发行|提供 6 个月的 Bug 修复支持，一年的安全修复支持|
|Laravel 5.5|2017 年 8 月|LTS 长久支持|Bug 修复 2019 年 8 月份，安全修复 2020 年 8 月份|


选择 Laravel 版本时，应该 优先考虑 **LTS 版本**，因为安全性和稳定性考虑，商业项目开发中 不应该 使用最新版本的 『Laravel 一般发行版』 。扩展阅读：[如何选择 Laravel 框架版本](https://laravel-china.org/topics/2595/how-to-select-the-laravel-framework-version)。

> 什么是LTS
> 长期支持 （英语：Long-term support，缩写：LTS）是一种软件的产品生命周期政策

请使用以下命令来创建指定版本的 Laravel 项目：

```
    composer create-project laravel/laravel project-name --prefer-dist "5.5.*"
```
