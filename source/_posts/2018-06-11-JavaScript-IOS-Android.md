---
title: JavaScript 区分IOS/安卓 跳转下载
date: 2018-06-11 15:39:17
categories: 前端
tags: [JavaScript, FrontEnd]
---

> 简单做一下前端判断,根据设备跳转不同的下载环境

<!-- more -->

```
    var u = navigator.userAgent;
    if(!!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)){
        window.location.href='https://itunes.apple.com/cn/app/App Store下载地址';
    }else {
        window.location.href='http://android.myapp.com/myapp/应用宝下载地址';
    }
```
