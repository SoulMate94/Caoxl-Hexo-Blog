---
title: Nginx 优化
date: 2018-01-22 15:00:33
categories: Nginx
tags: [Nginx, Linux, 服务器, 优化]
---

关于 Nginx 优化实际实践记录在这里。

下列配置可以放在 `nginx.conf` 的 `http` 块中，也可以放到虚拟主机的 `server` 块中。根据情况配置。

<!--more-->

# 启用压缩

```
    # 开启gzip
    gzip on;
    
    # 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
    gzip_min_length 1k;
    
    # gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明
    gzip_comp_level 2;
    
    # 进行压缩的文件类型 (javascript有多种形式 其中的值可以在 mime.types 文件中找到)
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png font/ttf font/otf image/svg+xml;
    
    # 是否在 http header 中添加 Vary: Accept-Encoding，建议开启
    gzip_vary on;
    
    # 禁用IE 6 gzip
    gzip_disable "MSIE [1-6]\.";
```


# 响应与超时

```
    sendfile        on;
    
    tcp_nopush     on;
    
    keepalive_timeout  65;
    
    client_max_body_size 50m; # 缓冲区代理缓冲用户端请求的最大字节数,可以理解为保存到>本地再传给用户
    
    client_body_buffer_size 256k;
    
    client_header_timeout 3m;
    
    client_body_timeout 3m;
    
    send_timeout 3m;
    
    proxy_connect_timeout 300s; # nginx跟后端服务器连接超时时间(代理连接超时)
    
    proxy_read_timeout 300s; # 连接成功后，后端服务器响应时间(代理接收超时)
    
    proxy_send_timeout 300s;
    
    proxy_buffer_size 64k; # 设置代理服务器（nginx）保存用户头信息的缓冲区大小
    
    proxy_buffers 4 32k; # proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
    
    proxy_busy_buffers_size 64k; # 高负荷下缓冲大小（proxy_buffers*2）
    
    proxy_temp_file_write_size 64k; # 设定缓存文件夹大小，大于这个值，将从upstream服务器传递请求，而不缓冲到磁盘
    
    proxy_ignore_client_abort on; # 不允许代理端主动关闭连接
```