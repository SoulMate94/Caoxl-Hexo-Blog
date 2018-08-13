---
title: Nginx 虚拟目录和虚拟主机的配置
date: 2018-01-22 11:56:21
categories: Nginx
tags: [Nginx, VirtualHost]
---

> 无论是在 Apache 上还是在 Nginx 上，虚拟主机和虚拟域名的配置都是我经常用到的功能。

# nginx.conf 配置文件的几个常用命令

nginx 配置文件主要分为六个区域：

- `main:` 全局设置
- `events:` nginx工作模式
- `http:` http设置
- `sever:` 主机设置
- `location:` URL 匹配
- `upstream:` 负载均衡服务器设置

下面，就以在 Windows 上使用 phpStudy 集成开发环境举例说明下 Nginx 的虚拟目录和虚拟主机是如何配置的：

<!--more-->

# Nginx 虚拟目录配置

> 通俗地讲，虚拟目录的意思就是浏览器上输入的 URL 不一定就代表网站在文件系统中的绝对路径，而是可以在硬盘中的任意指定位置。

比如在浏览器上访问的是 `http://localhost/test`，在网站根目录 `C:/htdocs/` 下不一定就有 `test` 这个目录，而是可以在其他位置, 比如 `d:/not_test`。

找到并打开 nginx.conf，然后在 `location ~ //.ht {...}` 字样下面添加即可：

```
    location /test {
         alias   "C:/Users/cjli/PhpstormProjects/test";
         index   index.php;
         autoindex  on;
    }
```

这里 `/test` 中的 test 是一个别名( alias )，可以自定义，而 alias 指令后面跟的路径也可以随意指向文件系统中任何存在的目录。

**重启 Nginx** 后打开 `http://localhost/test` 就可以看到上面目录 `C:/Users/caoxl/PhpstormProjects/test` 下的网站了( 如果有的话 )。

**注意:**

- 虚拟目录路径的配置不能用 `root` 指令而必须用 `alias` 指令。
- 路径必须用双引号括起来。
- `index` 指令和 `autoindex` 指令必须同时出现。

# Nginx 虚拟主机配置

通俗地讲，虚拟主机的意思就是，功能上力求和一台物理机实现得一致。

因此就很容易解释为什么多个虚拟主机中可以监听同一个端口号这样的问题了，因为虽然端口号是相同的，但是可以理解为是不同个主机上的同一个端口，这当然不影响了。 

为了管理方便，对 Nginx 的配置文件都可以放在一个文件夹下，然后统一在主配置文件 nginx.conf 中使用 include 指令包含进来。可参考如下：

- **conf.d/virtual-hosts.conf**

```
    server {
    	listen       80;
    	server_name  localhost 192.168.1.174 zh.oc.com;
    	charset utf-8;
    	location / {
    	    root       "D:\WWW\mycncart";
    	    index      index.php index.html index.htm;
    	    autoindex  on;
    	}
    	location ~ \.php(.*)$ {
    	    root       "D:\WWW\mycncart";
    	    fastcgi_pass   127.0.0.1:9000;
    	    fastcgi_index  index.php;
    	    fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
    	    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    	    fastcgi_param  PATH_INFO  $fastcgi_path_info;
    	    fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
    	    include        fastcgi_params;
    	}
    }
```

- **nginx.conf**

```
    # ...
    include conf.d/virtual-hosts.conf
```

**说明:**

- 每个虚拟主机在 nginx.conf 中都是一个单独的 server{} 块，配置思路也大体相同。
- **虚拟主机的端口可以监听同一个端口。**
- `server_name` 可以是内网IP、域名，公网 IP 和域名，也可以一次性指向多个域名或 IP。
- `root` 指令放在 `location` 指令在之外类似于全局变量，而每个 `location` 块中都可以使用该指令设置的路径。

在与 `PHP` 关联的时候，要么全局中指定过 `root`，否则在 `location` 中也必须指定根路径，否则重启 `Nginx` 后也无法找到文件，出现 `404` 或无法加载网页。

# 参考

- [nginx的配置、虚拟主机、负载均衡和反向代理](https://www.zybuluo.com/phper/note/89391)
- [windows下搭建nginx+php+虚拟主机配置过程](http://www.lxway.com/566945626.htm)
- [PHP-FastCGI on Windows](https://www.nginx.com/resources/wiki/start/topics/examples/phpfastcgionwindows/)