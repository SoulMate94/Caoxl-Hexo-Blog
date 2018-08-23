---
title: 「代码复用」获取客户端IP
date: 2018-08-08 10:51:36
categories: 代码复用
tags: [代码复用, Client, IP]
---

> Don't repeat yourself

<!-- more -->

# 直接上源码

```
    <?php
    
    // Client releated operations
    // @caoxl
    
    namespace App\Traits;
    
    class Client
    {
        public static function ip()
        {
            if ($clientIP = self::tryIPKey('HTTP_CLIENT_IP')) {
            } elseif ($clientIP = self::tryIPKey('HTTP_X_FORWARDED_FOR')) {
            } elseif ($clientIP = self::tryIPKey('HTTP_X_FORWARDED')) {
            } elseif ($clientIP = self::tryIPKey('HTTP_FORWARDED_FOR')) {
            } elseif ($clientIP = self::tryIPKey('HTTP_FORWARDED')) {
            } elseif ($clientIP = self::tryIPKey('REMOTE_ADDR')) {
            } else $clientIP = 'UNKNOWN';
    
            return $clientIP;
        }
    
        public static function tryIPKey(string $possibleKey)
        {
            return getenv($possibleKey)
                ?? (
                    $_SERVER[$possibleKey] ?? null
                );
        }
    }
```

**说明:**

- `getenv()` - 获取一个环境变量的值

- `HTTP_CLIENT_IP` - $_SERVER['HTTP_CLIENT_IP']

> HTTP_CLIENT_IP 是代理服务器发送的HTTP头。如果是“超级匿名代理”，则返回none值

- `HTTP_X_FORWARDED_FOR` - $_SERVER['HTTP_X_FORWARDED_FOR']

> 用来识别经过HTTP代理后的客户端IP地址（有可能存在，也可以伪造）

- `HTTP_X_FORWARDED` - $_SERVER['HTTP_X_FORWARDED']

- `HTTP_FORWARDED_FOR` - $_SERVER['HTTP_FORWARDED_FOR']

> X-Forwarded-For（XFF）是用来识别通过HTTP代理或负载均衡方式连接到Web服务器的客户端最原始的IP地址的HTTP请求头字段

- `HTTP_FORWARDED` - $_SERVER['HTTP_FORWARDED']

- `REMOTE_ADDR` - $_SERVER['REMOTE_ADDR']

> **REMOTE_ADDR** 是你的客户端跟你的服务器“握手”时候的IP。如果使用了“匿名代理”，REMOTE_ADDR将显示代理服务器的IP。 
> 浏览当前页面的用户的 IP 地址。