---
title: Nginx 反向代理
date: 2018-01-22 14:50:19
categories: Nginx
tags: Nginx
---

> 或名：不同域名通过一台服务器上的 80 端口访问同一台服务器上的不同站点。

# 环境和需求

- 一台 CentOS 主机上同时运行着 3 台服务器：Nginx、Apache、Tomcat，分别监听 80、8000、8080 端口，其中 Apache 托管了一个 PHP 站点 A，Tomcat 托管了一个 Java 站点 B。

- 现有域名 test.com 和 demo.com，需要实现的是：

在浏览器中输入 test.com 后直接(不加额外端口号) 访问站点 A；输入 demo.com 后直接访问站点 B。

<!--more-->

# 具体配置

- vi `/etc/nginx/nginx.conf`

不全，但是都是会改动的地方，不同的环境下的配置可能会有些许不同。参考如下：

```
    worker_rlimit_nofile 1024;
    events {
        use epoll;
        worker_connections  1024;
    }
    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /var/log/nginx/access.log  main;
        sendfile        on;
        #tcp_nopush     on;
        #keepalive_timeout  0;
        keepalive_timeout  65;
        gzip  on;
        
        # reverse proxy configuration changed by @cjli in 2016-03-13 14:50:21
    # ----------------------------------------------- start ----------------------------------------------
        client_max_body_size 50m; # 缓冲区代理缓冲用户端请求的最大字节数,可以理解为保存到本地再传给用户
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
    # ----------------------------------------------- end -----------------------------------------------
      # PHP conf here ...
      # Load config files from the /etc/nginx/conf.d directory
        include /etc/nginx/conf.d/*.conf;
    }
```

- vi `/etc/nginx/conf.d/reverse_proxy.conf`

由于 `nginx.conf` 的末尾已经有`include /etc/nginx/conf.d/*.conf` 了，因此可以在 `conf.d` 目录下创建不同需求的配置文件，这样方便管理。

*反向代理服务器的配置参考如下：*

```
    server
    {
        listen 80;
        server_name test.com;
        location / {
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8000;
        }
        access_log /var/log/nginx/test.log;
    }
    server
    {
        listen 80;
        server_name demo.com;
        location / {
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }
        access_log /var/log/nginx/demo.log;
    }
```

# 说明与注意事项

- **nginx -s reload 后报错？**

可能原因之一是：配置中指定的 `log` 路径不存在。


# 附录：`CentOS` 下 `Tomcat` 的配置

- **更改 Tomcat 默认端口**

Tomcat 默认监听的是 `8080`，在这里并不影响，但是目前没怎么配置过 `Linux` 下的 Tomcat，所以当做个笔记。

```
    find / -name server.xml
    vi /path/to/server.xml
```

找到 **Connector** 标签，更改其 port 属性值为所需端口号即可。


- **启动语关闭 Tomcat**

`CentOS/Linux` 下安装好 `Tomcat` 后，如果执行 `service tomcat start` 等相关命令如果提示找不到服务名的话，可能需要进入 Tomcat 的安装目录下的 `bin/` 目录手动执行：

```
    ./startup.sh
    ./shutdown.sh
    # ...
```

# 参考

- [搭建nginx反向代理用做内网域名转发](https://www.ttlsa.com/nginx/use-nginx-proxy/)