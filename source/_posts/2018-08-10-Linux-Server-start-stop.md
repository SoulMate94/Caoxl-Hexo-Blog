---
title: Linux 常见服务启动、重启、关闭命令
date: 2018-08-10 10:34:35
categories: Linux
tags: [Linux, Nginx, Apache, PHP-fpm]
---

> Linux 常见服务启动、重启、关闭命令

<!-- more -->

# Nginx

- 查看状态

```
    [root@caoxl ~]# service nginx status
    nginx (pid 1360 1359) is running...
```

- 启动

```
    [root@caoxl ~]# service nginx start
    Starting nginx... nginx: [warn] conflicting server name "caoxl.com" on 0.0.0.0:80, ignored
     done
    [root@caoxl ~]# service nginx status
    nginx (pid 14320 14319) is running...
```

- 重启

```
    [root@caoxl ~]# service nginx restart
    Stoping nginx... nginx: [warn] conflicting server name "caoxl.com" on 0.0.0.0:80, ignored
     done
    Starting nginx... nginx: [warn] conflicting server name "caoxl.com" on 0.0.0.0:80, ignored
     done
    [root@caoxl ~]# service nginx status
    nginx (pid 14354 14353) is running...
```

- 停止

```
    [root@caoxl ~]# service nginx stop
    Stoping nginx... nginx: [warn] conflicting server name "caoxl.com" on 0.0.0.0:80, ignored
     done
    [root@caoxl ~]# service nginx status
    nginx is stopped.
```


# Apache

```
    service httpd start     // 启动
    
    service httpd restart   // 重新启动
    
    service httpd stop      // 停止服务
```


# MySQL

```
    service mysql start      // 启动
    
    service mysql restart    // 重新启动
    
    service mysql stop       // 停止服务
```

# php-fpm

- 启动

```
    [root@caoxl ~]# service php-fpm start
    Starting php-fpm  done
    [root@caoxl ~]# service php-fpm status
    php-fpm (pid 14574) is running...
```

- 重启

```
    [root@caoxl ~]# service php-fpm restart
    Gracefully shutting down php-fpm . done
    Starting php-fpm  done
    [root@caoxl ~]# service php-fpm status
    php-fpm (pid 14602) is running...
```

- 停止

```
    [root@caoxl ~]# service php-fpm stop
    Gracefully shutting down php-fpm . done
    [root@caoxl ~]# service php-fpm status
    php-fpm is stopped
```

# 防火墙

```
    service iptables status       // 状态
    service iptables start        // 启动      
    service iptables stop         // 停止
    service iptables restart      // 重启
```

# 系统重启命令

- `shutdown`   
- `poweroff`  
- `reboot`  
- `halt`  
- `init`
