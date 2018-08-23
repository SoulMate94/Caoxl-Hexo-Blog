---
title: CSP 学习笔记
date: 2018-06-27 11:01:05
categories: Internet
tags: [Internet, CSP]
---

> CSP Is Dead, Long Live CSP! 
On the Insecurity of Whitelists and the Future of Content Security Policy

<!-- more -->

# CSP ?

> `Content Security Policy` 内容安全策略

# 诞生场景

能不能根本上解决跨域脚本攻击问题，浏览器自动禁止外部注入恶意脚本？

这就是"网页安全政策"（`Content Security Policy`，缩写 `CSP`）的来历

# `CSP`简介

> CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。
它的实现和执行全部由浏览器完成，开发者只需提供配置。


# 启用`CSP`

## 通过 `HTTP` 头信息的 `Content-Security-Policy` 的字段

```
    Content-Security-Policy: script-src 'self'; object-src 'none';
    style-src cdn.example.org third-party.org; child-src https:
```

## 通过网页的`<meta>`标签


```
    <meta http-equiv="Content-Security-Policy" content="script-src 'self'; object-src 'none'; style-src cdn.example.org third-party.org; child-src https:">
```

**说明:**

上面的代码中,`CSP`做了如下配置

- 脚本:   只信任当前域名
- <object>标签:   不信任任何URL, 即不加载任何资源
- 样式表:  只信任`cdn.example.org`和`third-party.org`
- 框架(frame):    必须适应`HTTPS`协议加载
- 其他资源: 没有限制

**启用后，不符合 CSP 的外部资源就会被阻止加载。**

# `CSP`限制选项

`CSP` 提供了很多限制选项，涉及安全的各个方面。

## 资源加载限制

- `script-src`: 外部脚本
- `style-src`:  样式表
- `img-src`:    图像
- `media-src`:  媒体文件(音频和视频)
- `font-src`:   字体文件
- `object-src`: 插件
- `child-src`:   框架
- `frame-ancestors`:    嵌入的外部资源
- `connect-src`:    `HTTP` 连接(通过XHR、WebSockets、EventSource等)
- `worker-src`: `worker`脚本
- `mainfest-src`:   `manifest`文件

## `default-src`

`default-src`用来设置上面各个选项的**默认值**。

```
    Content-Security-Policy: default-src 'self'
```

上面代码限制所有的外部资源，都只能从当前域名加载。

> 如果同时设置某个单项限制（比如`font-src`）和`default-src`，前者会覆盖后者，
即字体文件会采用`font-src`的值，其他资源依然采用default-src的值。

## `URL`限制

- `frame-ancestors`:    限制嵌入框架的网页
- `base-uri`:   限制<base#href>
- `form-action`:    限制<form#action>


## 其他限制

- `block-all-mixed-content`: HTTPS网页不得加载HTTP资源(浏览器已经默认开启)
- `upgrade-insecure-requests`: 自动将网页上所有加载外部资源的HTTP链接换成HTTPS协议
- `plugin-types`: 限制可以使用的插件格式
- `sandbox`: 浏览器行为的限制,比如不能有弹出窗口等

## `report-uri`

有时，我们不仅希望防止 XSS，还希望记录此类行为。`report-uri`就用来告诉浏览器，应该把注入行为报告给哪个网址。

```
    Content-Security-Policy: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
```

上面代码指定，将注入行为报告给`/my_amazing_csp_report_parser`这个 `URL`。

# `Content-Security-Policy-Report-Only`

除了`Content-Security-Policy`，还有一个`Content-Security-Policy-Report-Only`字段，
表示不执行限制选项，只是记录违反限制的行为。

它必须与`report-uri`选项配合使用。

```
    Content-Security-Policy-Report-Only: default-src 'self'; ...; report-uri /my_amazing_csp_report_parser;
```

# 选项值

每个限制选项可以设置以下几种值，这些值就构成了白名单。

- 主机名: exaple.org, https://example.com:443
- 路径名: example.org/resources/js/
- 通配符: *.example.org, *://*.example.com:* (表示任意协议、任意子域名、任意端口)
- 协议名: https、data
- 关键字'self': 当前域名,需要加引号
- 关键字'none': 禁止加载任何外部资源,需要加引号

多个值也可以并列，用空格分隔。

```
    Content-Security-Policy: script-src 'self' https://apis.google.com
```

如果同一个限制选项使用多次，只有第一次会生效。

```
   # 错误的写法
   script-src https://host1.com; script-src https://host2.com
   
   # 正确的写法
   script-src https://host1.com https://host2.com 
```

# `script-src` 的特殊值

除了常规值，`script-src` 还可以设置一些特殊值。注意，下面这些值都必须放在单引号里面。

- `unsafa-inline`: 允许执行页面内嵌的`&lt;script>`标签和事件监听函数
- `unsafe-eval`: 允许将字符串当做代码执行,比如使用`eval`,`setTimeout`,`setInterval`和`Function`等函数
- `nonce`值: 每次HTTP回应给出一个授权`token`, 页面内嵌脚本必须有这个token,才会执行
- `hash`值: 列出允许执行的脚本代码的`Hash`值,页面内嵌脚本的哈希值只有吻合的情况下,才会执行

# 注意

1. `script-src`和`object-src`是必设的，除非设置了`default-src`。

因为攻击者只要能注入脚本，其他限制都可以规避。而`object-src`必设是因为 `Flash` 里面可以执行外部脚本。

2. `script-src`不能使用`unsafe-inline`关键字（除非伴随一个`nonce`值），也不能允许设置`data:URL`。

举例:

```
    <img src="x" onerror="evil()">
    <script src="data:text/javascript,evil()"></script>
```

3. 必须特别注意 `JSONP` 的回调函数。

```
    <script
        src="/path/jsonp?callback=alert(document.domain)//">
    </script>
```

上面的代码中，虽然加载的脚本来自当前域名，但是通过改写回调函数，攻击者依然可以执行恶意代码。


# 参考

- [Content Security Policy 入门教程](http://www.ruanyifeng.com/blog/2016/09/csp.html)
- [内容安全策略( CSP )](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)
- [CSP Is Dead, Long Live CSP!](https://ai.google/research/pubs/pub45542)