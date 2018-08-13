---
title: Nginx 配置详解
date: 2018-06-26 10:06:02
categories: Nginx
tags: [Nginx]
---

> `Nginx`（发音同engine x）是一个异步框架的 `Web`服务器，也可以用作反向代理，负载平衡器 和 `HTTP`缓存.
> 作为一个后起之秀的**http服务器**，`nginx`与他的老大哥`apache`相比，在性能上，`nginx`占用更少的系统资源，
特定的场景应用(静态数据)能支持更多的并发连接，达到更高的访问效率；

<!-- more -->

# `Nginx` 配置管理

初始化的nginx默认主配置

执行 `grep -Evi "#|^$" nginx.conf` 得到如下结果！
> 或者直接找到 `nginx.conf` 打开即可

```
    worker_processes 1;
    
    events {
        worker_connections 1024;
    }
    
    http {
        include mime.types;
        default_type application/octet-stream;
        sendfile on;
        keepalive_timeout 65;
        server {
            listen 80;
            server_name localhost;
            location / {
                root html;
                index index.html index.htm
            }
            error_page 500 502 503 504 /50x.html;
            location = /50x.html {
                root html;
            }
        }
    }
```

# `Nginx` 配置段

## 全局配置区

- `worker_processes 1;`

指有一个工作的子进程，可以自行修改，因为要争`CPU`资源，一般设置为`CPU*核数`。

也可以设置成自动: `worker_processes auto;`

## `Event`配置区

一般是配置Nginx连接的特性

- `worker_connections 1024;`

指一个worker能同时允许产生多少个连接

> 配置值的时候需要结合系统参数

## `http`服务器配置区

### `server`虚拟主机配置区

> **提 示:** 如果未明确配置服务器IP地址访问配置，将使用第一个`server`配置。

#### 基于域名的虚拟主机

使用域名`www.caoxl.com`为例

```
    server {
        listen 80;
        server_name www.caoxl.com;
        error_page 404 /404.html;
    }
    
    location / {
        root html/www;
        index index.html;
        access_log logs/www.caoxl.com_access.log main;
        error_loh logs/www.caoxl.com_error.log;
    }
    
    location ~ .+\.php.*$ {
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index  index.php
        fastcgi_param  SCRIPT_FILENAME $document_root/$fastcgi_script_name;
        fastcgi_param  SCRIPT_FILENAME $request_filename;
        include  fastcgi_params;
    }
```

##### `Nginx` 日志管理

我们观察`Nginx`的`server`段，可以看到如下内容：

`access_log logs/www.caoxl.com_access.log main;`

- 参数1: 配置名
- 参数2: 指定访问日志的文件是 `logs/www.caoxl.com_access.log`
- 参数3: 使用的日志格式`main`格式, 同时我们也可以通过`log_format`配置其他格式的日志记录方式

```
    log_format main '$remote_addr - $remote_user [$time_local] "$request"'
                    '$status $body_bytes_sent "$http_referer"'
                    '"$http_user_agent" "$http_x_forwarded_for"';
```

另外和其他的web服务器一样，`nginx`可以针对不同的`server`配置不同的`log`

#### 基于端口的虚拟主机

以端口`8080`为例,域名依然使用`www.caoxl.com`

```
    server {
        listen 8080;
        server_name www.caoxl.com;
        error_page 404 /404.html;
        
        location / {
            root html/port;
            index index.html;
            access_log logs/www.caoxl.com_access.log main;
            error_log logs/www.caoxl.com.com_error.log;
        }
        
        location ~ .+\.php.*$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            include fastcgi_params;
        }
    }
```

#### 基于`IP`虚拟主机

以 `192.168.0.200` 为默认`IP`配置虚拟主机

```
    server {
        listen 80;
        server_name 192.168.0.200;
        error_page 404 /404.html;
        
        location / {
            root html/ipvhosts;
            index index.html;
            access_log logs/192.168.0.200_access.log main;
            error_log logs/192.168.0.200_error.log;
        }
        
        location ~ .+\.php.*$ {
            fastcgi_pass  127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
            fastcgi_param SCRIPT_FILENAME $request_filename;
        }
    }
```


# `Nginx`定时任务完成日志切割

任务需求：每日凌晨将`nginx`日志根据日期重命名日志文件进行切割。

- 昨天时间命令 `date -d yesterday +%Y%m%d`

```
    #!/bin/bash
    # filename cutlog.sh
    
    DATE=$(date -d yesterday + %Y%m%d)
    LOG_PATH=/usr/local/nginx/logs/
    LOG_NAME=access.log
    BASE_PATH=/var/log/
    SAVE_LOG_NAME=${DATE}.${LOG_NAME}
    
    mv ${LOG_PATH}${LOG_NAME} ${BASE_PATH}${SAVE_LOG_NAME}
    
    kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid)
```

## 定时执行任务

```
    1/* 0 * * * /usr/bin/crontab /root/cutlog.sh > /dev/null 2 > &1
```

# `Nginx`的`location`详解

## `location` 语法

```
    location [=|~|~*|^~\ patt {
    
    }
```

**参数解释：**

- `=` 开头表示精确匹配
- `^~`开头表示`uri`以某个常规字符串开头,理解为匹配`url`路径即可
> `nginx`不对`url`做编码，因此请求为`/static/20%/aa`，可以被规则`^~ /static/ /aa`匹配到（注意是空格）。
- `~`  开头表示区分大小写的正则匹配
- `~*` 开头表示不区分大小写的正则匹配
- `!~` 开头表示区分大小写不匹配的正则
- `!~*` 开头表示不区分大小写不匹配的正则

## `location`匹配类型

### `location`匹配之精准匹配

```
    location = / {
        # 规则A
    }
    
    location = /login {
        # 规则B
    }
```

### `location`匹配之正则匹配

```
    location ^~ /static/ {
        # 规则C
    }
    
    location ~ \.(gif|jpg|png|js|css)$ {
        # 规则D
    }
    
    location ~* \.png$ {
        # 规则E
    }
    
    location !~ \.xhtml$ {
        # 规则F
    }
    
    location !~* \.xhtml$ {
        # 规则G
    }
```

### `location`匹配之一般匹配

```
    location / {
        # 规则H
    }
```

### `location`配置优先级

1. 先判断精准匹配，如果匹配，立即返回结果并结束解析过程
2. 然后，判断普通命中没如果有多个命中，**记录最长**的匹配结果
3. 再然后判断正则表达式的解析过程，按配置里的正则表达式顺序为准，由上到下开始匹配，一旦匹配成功立即返回结果并结束解析过程。

通过上面的分析我们可以知道：

- 普通匹配与顺序无关，因为**按照匹配的长短来取匹配结果**。
- 正则匹配与顺序有关，因为是从上往下匹配。(首先匹配，取其之。结束解析过程)


# `nginx`的`rewrite`语法详解

## `rewrite` 使用环境

- `Server`
- `location`
- `if`

## 重写过程中可能用到的指令

```
    if (条件) {
        // 设定条件再进行重写
    }
    
    set # 设置变量
    
    return # 返回状态码
    
    break # 跳出rewrite
    
    rewrite # 重写
```

# `nginx`缓存`expires`模块

控制图片等过期时间为`30`天，当然这个时间可以设置的更长。
具体视情况而定,将下列的`location`配置写入`server`下。

```
    location ~\.(gif|jpg|jpeg|png|bmp|ico)$ {
        expires 30d;
    }
```


# `nginx.conf`配置文件

`Nginx`配置文件主要分成四部分：
- `main`（全局设置）、
- `server`（主机设置）、
- `upstream`（上游服务器设置，主要为反向代理、负载均衡相关配置）
- `location`（URL匹配特定位置后的设置）

> main部分设置的指令将影响其它所有部分的设置；
> server部分的指令主要用于指定虚拟主机域名、IP和端口；
> upstream的指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡；
> location部分用于匹配网页位置（比如，根目录“/”,“/images”,等等）
> 他们之间的关系式：server继承main，location继承server；upstream既不会继承指令也不会被继承。它有自己的特殊指令，不需要在其他地方的应用。

## 通用

下面的`nginx.conf`简单的实现`nginx`在前端做反向代理服务器的例子，
处理`js`、`png`等静态文件，`jsp`等动态请求转发到其它服务器`tomcat`：

```
    # nginx.conf
    
    user www www;
    worker_processes 2;
    
    error_log logs/error.log;
    #error_log logs/error.log notice;
    #error_log logs/error.log info;
    
    pid  logs/nginx.pid;
    
    events {
        use epoll;
        worker_connections 2048;
    }
    
    http {
        include       mime.types;
        default_type  application/octet-stream;
        
        #log_format   main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
        
        #access_log   logs/access.log main;
        
        sendfile      on;
        # tcp_nopush  on;
        
        keepalive_timeout  65;
        
        #gzip压缩功能设置
        gzip on;
        gzip_min_length 1k;
        gzip_buffers    4 16k;
        gzip_http_version 1.0;
        gzip_comp_level 6;
        gzip_types text/html text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
        gzip_vary on; 
        
        #http_proxy设置
        client_max_body_size   10m;
        client_body_buffer_size   128k;
        proxy_connect_timeout   75;
        proxy_send_timeout   75;
        proxy_read_timeout   75;
        proxy_buffer_size   4k;
        proxy_buffers   4 32k;
        proxy_busy_buffers_size   64k;
        proxy_temp_file_write_size  64k;
        proxy_temp_path   /usr/local/nginx/proxy_temp 1 2;
        
        #设定负载均衡后台服务器列表
        upstream backend {
            #ip_hash;
            server 192.168.10.100:8080 max_fails=2 fail_timeout=30s ;
            server 192.168.11.100:8080 max_fails=2 fail_timeout=30s ;
        }
        
        #很重要的虚拟主机配置
        server {
            listen  80;
            server_name itoatest.example.com;
            root /apps/oaapp;
            
            charset utf-8;
            access_log logs/host.access.log main;
            
            #对 / 所有做负载均衡+反向代理
            location / {
                root /apps/oaapp;
                index index.jsp index.html index.htm;
                
                proxy_pass  http://backend;
                proxy_redirect off;
                
                #后端的web服务器可以通过X-Forward-For获取用户真实IP
                proxy_set_header  Host  $host;
                proxy_set_header  X-Real-IP  $remote_addr;  
                proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
                proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            }
            
            #静态文件，nginx自己处理，不去backend请求tomcat
            location  ~* /download/ {  
                root /apps/oa/fs;  
    
            }
            location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$   
            {   
                root /apps/oaapp;   
                expires      7d; 
            }
            location /nginx_status {
                stub_status on;
                access_log off;
                allow 192.168.10.0/24;
                deny all;
            }
    
            location ~ ^/(WEB-INF)/ {   
                deny all;   
            }
            #error_page  404              /404.html;
    
            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
        }
        ## 其它虚拟主机，server 指令开始
    }
```

# 参考

- [nginx服务器安装及配置文件详解](https://segmentfault.com/a/1190000002797601)