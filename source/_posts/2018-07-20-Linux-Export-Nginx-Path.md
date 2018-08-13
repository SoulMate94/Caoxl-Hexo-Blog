---
title: 将Nginx加到系统变量中
date: 2018-07-20 16:07:34
categories: Linux
tags: Linux
---

> `PHP`, `MYSQL`和这个方法一样

<!-- more -->

# 配置环境变量

- 在`/etc/profile`中加入

```
    export NGINX_HOME=/usr/local/nginx
    export PATH=$PATH:$NGINX_HOME/sbin
```

执行 `source /etc/profile`, 使配置文件生效即可.