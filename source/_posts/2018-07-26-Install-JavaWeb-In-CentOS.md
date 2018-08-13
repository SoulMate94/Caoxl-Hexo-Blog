---
title: CentOS下配置安装JavaWeb环境
date: 2018-07-26 14:44:41
categories: Linux
tags: [Linux, JavaWeb]
---

> 搭建 `Java` 开发环境

<!-- more -->

# 安装 `JDK`

- `JDK` 是开发`Java`程序必须安装的软件,我们查看一下 `yum` 源里面的 `JDK`:

```
    yum list java*
    
    # 或
    yum list | grep jdk
```

- 选择适合本机的`JDK`，并安装:

```
    yum install java-1.8.0-openjdk* -y
```

- 安装完成后，查看是否安装成功(安装成功回显示jdk的版本号):

```
    [root@caoxianliang local]# java -version
    openjdk version "1.8.0_171"
    OpenJDK Runtime Environment (build 1.8.0_171-b10)
    OpenJDK 64-Bit Server VM (build 25.171-b10, mixed mode)
```

# 安装`Tomcat`

> 下载[Tomcat](http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.10/bin/apache-tomcat-9.0.10.tar.gz)，并解压到指定目录中。

- `/usr/local/java-tomcat` (我的安装目录)

```
    cd /usr/local/java-tomcat
    tar zxvf apache-tomcat-9.0.10.tar.gz
    mv ./apache-tomcat-9.0.10/ /usr/local/tomcat9
```

- 修改权限

```
    chmod -R 777 /usr/local/tomcat9/bin
```

- 启动`tomcat`

```
    /usr/local/tomcat9/bin/startup.sh
```

- 修改`Nginx`配置 (`/usr/local/nginx/conf/nginx.conf`)

```
    location ~ .+\.jsp($|/) {
                    proxy_pass http://127.0.0.1:8080; #转发端口
                    proxy_set_header Host $host;
                    client_max_body_size 10m;
                    client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数。
                    proxy_connect_timeout 90; #Nginx跟后端服务器连接超时时间。
                    proxy_read_timeout 90; #连接成功后，后端服务器响应时间。
                    proxy_buffer_size 4k; #设置代理服务器保存用户头信息的缓冲区大小。
                    proxy_buffers 6 32k; #proxy_buffers缓冲区。
                    proxy_busy_buffers_size 64k; #高负荷下缓冲大小。
                    proxy_temp_file_write_size 64k;#设定缓存文件夹大小。
            }
```

- `java.com.conf`

```
    server {
        listen       80;
        #listen       [::]:80 default_server;
        server_name  java.caoxl.com;
    
        # Load configuration files for the default server block.
        #include /etc/nginx/default.d/*.conf;
    
        location /i/ {
            alias /usr/local/tomcat9/webapps/ROOT/;
    }
    
    location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-Ip $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass   http://127.0.0.1:8080;
    }
    
        error_page 404 /404.html;
            location = /40x.html {
        }
    
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    
    }
```

**别忘了 `service nginx restart` 重启`Nginx`**

> 访问: [java.caoxl.com](http://java.caoxl.com/index.jsp) 即可看到

# 动静分离

- 新建一个`location`来专门转发静态文件

```
    location ~ \.(html|js|css|gif|jpg|png|bmp|swf)$ {
        expires 30d;
        root /usr/local/tomcat9/webapps/ROOT;
    }
```

> 至此, `JavaWeb`搭建完成, 最后说一句, `PHP`是世界上最好的语言 