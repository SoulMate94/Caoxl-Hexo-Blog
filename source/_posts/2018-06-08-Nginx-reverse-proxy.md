---
title: Nginx 反向代理/负载均衡
date: 2018-06-08 15:38:49
categories: Nginx
tags: [Nginx]
---

> 反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，
然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，
此时站在服务器角度来看，代理服务器对外就表现为一个反向代理服务器。

<!-- more -->

# `Nginx`负载均衡集群配置

- 修改`nginx.conf`配置文件
- 配置服务器组,在`http{}`节点之间添加`upstream`配置(注意不要写`localhost`，不然访问速度会很慢)

```
    upstream nginx {
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
    }
```

- 修改`nginx`监听的端口号`80`, 改为`8080`

```
    listen 8080;
```

- 在`location / {}`中,利用`proxy_pass`配置反向代理地址;此处`"http://"`不能少,
后面的地址要和第一步`upstream`的名称保持一致.

```
    location / {
        proxy_pass http://nginx; #配置反向代理地址
    }
```

访问:url:http://127.0.0.1:8080/nginx


# `Nginx`负载均衡策略

## 一、轮询（默认）

每个web请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

```
    upstream nginxDemo {
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
    }
```

## 二、最少链接

web请求会被转发到连接数最少的服务器上。

```
    upstream nginxDemo {
        least_conn;
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
    }
```

## 三、`weight`权重

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况，weight默认是1。

```
   #服务器A和服务器B的访问比例为：2-1;比如有3个请求，前两个会访问A，三个访问B，其它规则和轮询一样。
   upstream nginxDemo {
        server 127.0.0.1:8081 weight=2; #服务器A
        server 127.0.0.1:8082; #服务器B
   } 
```

## 四、`ip_hash`

每个请求按访问`ip`的`hash`值分配，这样同一客户端连续的Web请求都会被分发到同一服务器进行处理，
可以解决`session`的问题。当后台服务器宕机时，会自动跳转到其它服务器。

```
    upstrea nginxDemo {
        ip_hash;
        server 127.0.0.1:8081 weight=2; #服务器A
        server 127.0.0.1:8082; #服务器B
    }
```

> 基于`weight`的负载均衡和基于`ip_hash`的负载均衡可以组合在一起使用。

## 五、`url_hash (第三方)`

`url_hash`是nginx的第三方模块，nginx本身不支持，需要打补丁。

`nginx`按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存服务器、文件服务器、静态服务器时比较有效。

缺点是当后端服务器宕机的时候，url_hash不会自动跳转的其他缓存服务器，而是返回给用户一个503错误。

```
    upstream nginxDemo {
        server 127.0.0.1:8081; #服务器A
        server 127.0.0.1:8082; #服务器B
        hash $requert_url;
    }
```

## 六、`fair (第三方)`

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```
    upstream nginxDemo {
        server 127.0.0.1:8081; #服务器A
        server 127.0.0.1:8082; #服务器B
        fair;
    }
```