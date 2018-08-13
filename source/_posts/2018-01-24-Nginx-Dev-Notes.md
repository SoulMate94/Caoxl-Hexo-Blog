---
title: Nginx 「通用日志」
date: 2018-01-24 16:30:25
categories: Nginx
tags: [Nginx, 配置]
---

Nginx 能做的事实在太多，先遇到什么记录下什么吧。

和本博客其他很多汇总性日志一样，本文内容也会不定期更新。

> <u>http://nginx.org/en/docs/</u>

<!--more-->

# 泛域名配置

## 自动关联子域名到目录名

```
    server {
      # ...
      server_name ~^(?<subdomain>.+).example.com$;
      root /data/wwwroot/example.com/$subdomain;
      # ...
    }
```

如果 nginx 后面是 PHP，只是这样配置会出现 `$_SERVER['host']` 一字不差的为 `~^(?<subdomain>.+).example.com$`，因此还需要添加一个 `fastcgi_param` 的 `SERVER_NAME` 配置：

```
    location ~ [^/]\.php(/|$) {
      #fastcgi_pass remote_php_ip:9000;
      fastcgi_pass unix:/dev/shm/php-cgi.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
      fastcgi_param SERVER_NAME $host;    # 重新赋值 SERVER_NAME 参数覆盖前面 `server_name` 的初始定义
    }
```

# 安全

## IP 访问频率限制

```
    geo $whitelist {
       default 0;
       # CIDR in the list below are not limited
       1.2.3.0/24 1;
       9.10.11.12/32 1;
       127.0.0.1/32 1;
    }
    
    map $whitelist $limit {
        0     $binary_remote_addr;
        1     "";
    }
    
    # 2. The directives below limit concurrent connections from a 
    # non-whitelisted IP address to five
    limit_conn_zone      $limit    zone=connlimit:10m;
    limit_conn           connlimit 5;
    limit_conn_log_level warn;   # logging level when threshold exceeded
    limit_conn_status    503;    # the error code to return
    
    # 3. Limits the number requests from a non-whitelisted IP to
    # one every two seconds with up to 3 requests per IP delayed
    # until the average time between responses reaches the threshold. 
    # Further requests over and above this limit will result 
    # in an immediate 503 error
    limit_req_zone       $limit   zone=one:10m  rate=30r/m;
    limit_req            zone=one burst=3;    # burst => 超过请求频率限制后最多允许多少次强制性访问
    limit_req_log_level  warn;
    limit_req_status     503;
```

配置好后可以简单通过命令来测试结果：`ab -n 5000 -c 100 http://www.example.com/`，会显示绝大部分请求均返回 `503`。

**注** ab是做压力测试的一种方法,如果显示没有这个命令,运行

```
    yum install httpd-tools
```

**配置说明：**

- `geo` 定义了白名单 `$whitelist`：默认值为 0；如果请求 IP 与白名单中的 IP 匹配，则`$whitelist` 值为 1
- map 指令是将 `$whitelist` 值为 0 的，也就是受限制的 IP，映射为请求 IP。将 `$whitelist` 值为 1 的，也就是白名单 IP，映射为空的字符串。
- `limit_conn_zone` 和 `limit_req_zone` 将忽略键值为空请求 IP，从而实现对白名单中的 IP 不限制而对其他 IP 做限制。

## 设置 nginx 密码认证

```
    location /path {
      auth_basic "Restricted";
      auth_basic_user_file /path/to/nginx/.htpasswd;
    }
```

## IP 黑／白名单

```
    location /path {
      allow whitelist;    # whitelist 事先定义过
      deny all;
    }
```

被 `deny` 掉的 IP 会得到 `403` 状态码。可以制定一个拒绝访问的页面：

```
    location / {
      error_page 403 = @deny;
      allow whitelist;
      deny all;
    }
    
    localtion @deny {
      return 301 http://site.com/cominglater;
    }
```

### `allow` 和 `deny` 的顺序问题

- 先 `allow` 部分后 `deny all`，表示只允许部分 IP 访问
- 先 `deny all` 后 `allow` 部分，表示拒绝全部

### 禁止指定 IP 或 IP 段

```
    # /path/to/blacklist.ip
    deny 192.168.33.111;
    deny 192.168.44.222;
    deny 192.168.55.0/24;    # 禁止 192.168.10.1~192.168.10.254
    
    --------------------------
    
    # /path/to/nginx.conf
    include /path/to/blacklist.ip    # 绝对路径
    # include blacklist.ip    # 相对于 nginx.conf 路径
```

## 限制特定目录访问

```
    localtion ~ .*(config|cache)/.*\.php$ {
      deny all;
    }
```

## 限制 UA

```
    location / {
      if ($http_user_agent ~ * 'ApacheBench|bingbot/2.0|Spider/3.0') {
        return 403;
      }
    }
```

# Rewrite 规则

## 别忘了限制条件

```
    if (!-e $request_filename) {
      rewrite ^(.*)$ /index.php last;
    }
```

### 如果不在限制条件下使用 `rewrite` 规则，容易造成死循环。

- HTTP 重定向到 HTTPS

```
    server {
      listen 80;
      server_name www.example.com;
      rewrite ^/(.*) https://$server_name$1 permanent;   # 301
      # rewrite ^/(.*) https://$server_name$1 redirect;    # 302
    }
```

## PHP 项目常用 rewrite 规则

- Laravel

```
    location / {
      try_files $uri $uri/ /index.php?$query_string;
    }
```

- 禅道项目管理系统

```
    server {
        listen       80;
        server_name  zentao.example.com;
        access_log /data/wwwlogs/zentao.example.com_nginx.log combined;
        index index.html index.htm index.php;
        root /data/wwwroot/zentao.example.com/www;
        charset utf-8;
        location / {
            try_files $uri /index.php$uri;
        }
        location ~ \.php(/.+)?$ {
            fastcgi_pass unix:/dev/shm/php-cgi.sock;
            fastcgi_index index.php;
            include fastcgi.conf;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }
    }
```

# 日志配置

## 关闭对 favicon 和 robot.txt 的日志记录

```
    location = /favicon.ico {
      access_log off;
      log_not_found off;
    }
    location = /robot.txt {
      access_log off;
      log_not_found off;
    }
```

## 只记录特定状态的访问日志

```
    http {
        map $status $normal {
            ~^2  1;
            default 0;
        }
        map $status $abnormal {
            ~^2  0;
            default 1;
        }
        map $remote_addr $islocal {
            ~^127  1;
            default 0;
        }
      
        server {
            access_log logs/access.log combined if=$normal;
            access_log logs/access_abnormal.log combined if=$abnormal;
            access_log logs/access_local.log combined if=$islocal;
        }  
    }
```

# With PHP

## `nginx` 与 `fastcgi／php-fpm` 之间的关系

- Nginx：Web 服务器。
- FastCGI：同 `cgi` 一样只是一个协议，负责规范 Web 应用程序和 Web 服务器的连接，规定了传递给 PHP 等动态程序的有哪些参数以及通过什么形式传递。
- <u>[php-fpm](http://php.net/manual/zh/install.fpm.php)</u>：PHP 官方采纳并使用的，实现了 `FastCGI` 协议的进程管理器，负责管理和调度 `php-cgi` 进程。

> 这里的「Web 应用程序」指的是 PHP 开放的应用程序，虽然我们常说 PHP 服务器后端开发，但是站在 Nginx 角度，PHP 开发的后端程序依然是应用程序的范畴。

`HTTP` 请求到达服务器之后，首先经过专门的 `Web` 服务器，然后 `Web` 服务器会将该请求简单处理后将按照 `CGI／FastCGI` 规定的标准数据格式交给 `PHP-FPM` 处理。

php-fpm 的安装，旧版本 PHP 中需要下载对应 PHP 版本的 php-fpm 源码以内核补丁的形式编译到 PHP 中，PHP 5.4 之后 PHP 核心已内置 `php-fpm`，只需在安装的时候启用 `--enable-fpm` 就行了。


## nginx+fpm 高并发优化思路

- 启用 `opcache`（no zend guard loader）
- `nginx` 和 `fpm` 的通信方式使用 `UNIX socket` 替换 `TCP` 端口

```
    fastcgi_pass 127.0.0.1:9000;
    # 换为：
    fastcgi_pass unix:/dev/shm/php-fpm.sock;
```

其中，`/dev/shm/php-fpm.sock` 是在 `php-fpm.conf` 中配置的。

- 优化 Linux 内核参数

修改 */etc/sysctl.conf*（_/proc/sys/net/core/_somaxconn）后使用 `sysctl -p` 重启：

```
    net.ipv4.tcp_max_syn_backlog = 16384
    net.core.netdev_max_backlog = 16384 
    net.core.somaxconn = 2048
```

- 优化 `php-fpm` 参数

```
    listen.backlog = 8192;     ; 默认为 -1（由系统决定）
    listen 80 backlog = 8192;  ; 默认为 511
```

# 反向代理

```
    upstream service_a {
      server 10.10.10.100:80;
      server 10.10.10.200:80;
    }
    server {
      listen 80;
      server_name _;
      location / {
        proxy_pass http://service_a/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
    }
```

# FAQ

## 客户端获得 403 原因

> “403” is actually an HTTP status code that means that the web server has received and understood your request, but that it cannot take any further action.

- 客户端访问目录而 `autoindex` 设置成 `off`。

  由于安全原因，目录索引默认是关闭的。打开方式：
  
  ```
    location /path {
      autoindex on;
      autoindex_exact_size_off;
    }
  ```
  
  `autoindx` 必须出现在 `location` 块中。

- 客户端访问了只能在服务端内部访问的资源。

- nginx 无需要的网站路径权限

文件要被能 `nginx read`，目录要能被 `nginx execute`。

- 目录索引文件未定义

以 PHP 为例，如果只定义 `index index.html index.htm` 则 nginx 会直接返回 403 而无论是否真实存在 `index.php`。

- 索引定义的文件名大小写弄错

`index` 指令使大小写区分的，如果 `index.php` 错写成 `Index.php` 依然会出现 `403`.

- `nginx 屏蔽了指定的客户端／IP`，通常通过 deny 等指令设置。


# 参考

- [Module ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)

- [How to rate-limit in nginx, but including/excluding certain IP addresses?](https://serverfault.com/questions/177461/how-to-rate-limit-in-nginx-but-including-excluding-certain-ip-addresses)

- [nginx定制header返回信息模块ngx_headers_more](http://www.ttlsa.com/nginx/nginx-custom-header-to-return-information-module-ngx_headers_more/)

- [Nginx下禅道的伪静态(rewrite)规则](http://yaxin-cn.github.io/Nginx/configure-static-access-for-zentao-under-nginx.html)

- [Nginx Access Log日志统计分析常用命令](http://www.cnblogs.com/coolworld/p/6726538.html)

- [nginx防止DDOS攻击配置（2）](https://www.52os.net/articles/nginx-anti-ddos-setting-2.html)

- [How to Secure Nginx Using Fail2ban on Centos-7]](https://hostpresto.com/community/tutorials/how-to-secure-nginx-using-fail2ban-on-centos-7/)

- [nginx通过云负载均衡后作反向代理并限制制定ip访问](http://www.voidcn.com/article/p-durjqmpt-bro.html)

- [IP.cn - 每日中国大陆 IP 列表, 电信 IP 列表，联通 IP 列表，移动 IP 列表](ip.cn)