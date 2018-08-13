---
title: 关于如何禁止谷歌浏览器强制转 HTTPS 
date: 2018-03-29 17:31:14
categories: www
tags: www
---

Chrome 于 V63 版本起会将 `.dev` 域名强制转换为 HTTPS 

<!-- more -->

看了很多文章都没有什么实际作用，最后发现不适用 `.dev` 就解决了


简单来说 建议您使用以下域名开发：

- .localhost
- .invalid
- .test
- .example


开发环境下改域名后缀就好了。至少我是这么解决的。
