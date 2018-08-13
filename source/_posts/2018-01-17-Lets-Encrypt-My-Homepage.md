---
title: Let's Encrypt 简单使用
date: 2018-01-17 14:37:53
categories: www
tags: [HTTPS, Let's Encrypt, 服务器]
---

## 初次部署 https

> 使用 <u>[Let's Encrypt](https://letsencrypt.org/)</u>给域名加把小绿锁

### Let’s Encrypt 客户端

### 安装
```
    # yum install -y epel-release nginx git
    
    git clone https://github.com/letsencrypt/letsencrypt
    
    cd letsencrypt
    
    ./letsencrypt-auto --help
```

<!--more-->

### 获取 SSL 证书

#### 证书申请频率限制
 - IP 限制：每注册 IP 每 3 个小时不超过 10 次
 - 域名数量限制：每个域名（包含子域名）每 7 天不超过 5 个
 
Let’s Encrypt 验证方式有很多种，我使用的服务器已经运行了 `nginx` 且不想终止 web 服务，因此使用 `webroot` 模式。

#### webroot 模式
```
    ./letsencrypt-auto certonly --webroot --webroot-path /home/wwwroot/default -d caoxl.com --agree-tos --email code0809@163.com
```

**注意: `/home/wwwroot/defaul` 是项目根目录**



#### certonly 模式

当然，除了 `webroot` 模式，也可以是用先手动只获取证书，然后配置 `nginx`。

```
    service nginx stop
    # Or: systemctl stop nginx
    
    ss -tln
    ./letsencrypt-auto certonly --standalone -d caoxl.com
```

唯一需要注意的一点就是，`certonly` 模式下获取证书需要先关闭 nginx。

获取成功后，会输出类似如下内容：

```
    Saving debug log to /var/log/letsencrypt/letsencrypt.log
    Plugins selected: Authenticator standalone, Installer None
    Obtaining a new certificate
    Performing the following challenges:
    tls-sni-01 challenge for caoxl.com
    Waiting for verification...
    Cleaning up challenges
    IMPORTANT NOTES:
     - Congratulations! Your certificate and chain have been saved at:
       /etc/letsencrypt/live/caoxl.com/fullchain.pem
       Your key file has been saved at:
       /etc/letsencrypt/live/caoxl.com/privkey.pem
       Your cert will expire on 2017-05-08. To obtain a new or tweaked
       version of this certificate in the future, simply run
       letsencrypt-auto again. To non-interactively renew *all* of your
       certificates, run "letsencrypt-auto renew"
     - If you like Certbot, please consider supporting our work by:
       Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
       Donating to EFF:                    https://eff.org/donate-le
```

然后，注意上面成功提示信息中的证书和私钥文件的路径，用于接下来 nginx 的配置。

- 证书: `/etc/letsencrypt/live/caoxl.com/fullchain.pem`
- 私钥文件: `/etc/letsencrypt/live/caoxl.com/privkey.pem`

### 生成 dhparam

为了进一步提高安全性，建议为 nginx 再生成 2048 位 `DH parameters`：

```
    openssl dhparam -out /etc/ssl/certs/dhparams.pem 2048
```

关于 [DH Parameters](https://wiki.openssl.org/index.php/Diffie-Hellman_parameters) 的解释，可以参考：<u>[What’s the purpose of DH Parameters?](https://security.stackexchange.com/questions/94390/whats-the-purpose-of-dh-parameters)</u>。

### nginx 部署 ssl 证书

#### 为域名配置 443 端口的虚拟主机

> 我的LNMP环境, 打开 /usr/local/nginx/conf/vhost/caoxl.com.conf

```
    # caoxl.com.conf
    server {
      listen 80;
      listen 443 ssl http2;
      server_name caoxl.com www.caoxl.com;
      ssl_certificate /etc/letsencrypt/live/caoxl.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/caoxl.com/privkey.pem;
      ssl_dhparam /etc/ssl/certs/dhparams.pem;
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
      ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
      ssl_prefer_server_ciphers on;
      ssl_session_timeout 10m;
      ssl_session_cache builtin:1000 shared:SSL:10m;
      ssl_buffer_size 1400;
      add_header Strict-Transport-Security max-age=15768000;
      ssl_stapling on;
      ssl_stapling_verify on;
      server_name caoxl.com www.caoxl.com;
      location / {
        root   caoxl.com;
        index  index.html index.htm;
      }
    }
```

#### http 强跳转 https（为同样的域名配置 80 端口的虚拟主机）
```
    # caoxl.com.conf
    server {
        listen 80;
        server_name caoxl.com www.caoxl.com;
        return 301 https://caoxl.com;
    }
```

#### 重载配置后测试
```
    nginx -t
    nginx -s reload
```

然后访问 <u>https://caoxl.com</u> 和 <u>http://caoxl.com</u>，不出意外应该就能在浏览器地址栏最左端看到一把靠谱的小绿锁了。

**<u>[点击此处输入域名查询SSL证书](https://www.ssllabs.com/ssltest/index.html)</u>**

## 证书续期

由于 Let’s Encrypt 颁发的服务器证书有效期只有 90 天，因此如果需要长期使用，就有必要设置好自动续期。

通过 letsencrypt-auto 工具，手动续期命令为：
```
    ./letsencrypt-auto renew
```

而所谓的自动续期，就是自动定时执行上面手动获得证书的操作，对于 Linux 来说，可以使用 crontab。

```
    echo '@monthly root /path/to/letsencrypt-auto certonly --webroot --webroot-path /home/wwwroot/default -d caoxl.com --agree-tos --email code0809@163.com >> /var/log/letsencrypt/letsencrypt-auto-update.log' | tee --append /etc/crontab
```

## 其他姿势

### <u>[acme.sh](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)</u>

以 **nginx** 为例：
```
    # 0. 安装 acme.sh 
    curl https://get.acme.sh | sh
    # 1. 获取证书
    acme.sh --issue -d www.example.com -w /path/to/webapp
    # 2. 生成 dhparam
    # 同上，略
    # 3. 拷贝证书到 WEB 服务器能正常读取的路径 (安装证书)
    acme.sh  --installcert  -d  example.com   \
    --key-file   /usr/local/nginx/conf/ssl/example.com.key \
    --fullchain-file /usr/local/nginx/conf/ssl/fullchain.cer \
    # --reloadcmd  "service nginx force-reload"    # 可选，我是安装证书完成后，手动改完配置再重启的
    # 4. 配置 nginx
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    listen 443 ssl;
    ssl_certificate         /path/to/ssl/example.com.cer;
    ssl_certificate_key     /path/to/ssl/example.com.key;
    ssl_dhparam             /path/to/ssl/dhparam.pem;
    # 5. 强制重启 nginx
    nginx -t
    nginx -s reload    # 如果证书配置不能被加载成功，请强制性重启 nginx
```

## FAQ
 - **已经成功安装配置好证书，部分 URL 仍然显示非安全？**
 
 显示非安全的 URL 页面里面含有 mixed content。
 > When you set a site to use SSL, all the contents, urls to load js, images, etc. should have https links instead of http.
 
 **简单来说:就是网站内包含不安全内容**
 
 - **使用 oneinstack 新建 vhost 时安装 let‘s Encrypt 提示：DNS problem: NXDOMAIN looking up A**
 
 这可能是因为在域名解析设置那里，给这个域名设置了 CDN 的 CNAME（出于加速静态文件和隐藏服务器真实地址的原因），而阿里云不能给相同主机头（RR）同时设置一个 CNAME 和 A 纪录，导致 Let’s Encrypt 检测不通过，只能删除 CNAME 纪录。
 
 - **To fix these errors, please make sure that your domain name was entered correctly and the DNS A/AAAA record(s) for that domain contain(s) the right IP address.**
 
 查看域名解析中 `A` 记录中的IP地址是否指向自己的服务器IP地址,且只能指向自己的服务器IP地址,不可多个指向.
 
 > 我刚开始一直遇到这个问题就是因为的我A记录指向了两个不同的IP
 
 此外，对于所申请 Let’s Encrypt 证书的域名来说，其顶级域名对应的 www 子域名必须要有一个 A 记录。可参考 LE 的该社区讨论：[No WWW Record in DNS Means Certficate Cannot Be Issued。](https://community.letsencrypt.org/t/no-www-record-in-dns-means-certficate-cannot-be-issued/14830/3)
 
 - **Problem binding to port 443: Could not bind to IPv4 or IPv6.. Skipping**
 执行证书续期命令的时候先，暂停 nginx，续期成功后再启动 nginx 即可。
 
## 参考

 - [如何获得Let’s Encrypt证书？Apache+Ubuntu](https://www.binarization.com/archive/2016/lets-encrypt-apache-ubuntu/)
 - [Let’s Encrypt SSL证书配置](https://www.jianshu.com/p/eaac0d082ba2)
 - [SSL Server Test 或者叫检测网站的证书](https://www.ssllabs.com/ssltest/index.html)
 - [oneinstack自动部署Lets Encrypt实现多域名SSL](http://www.ptbird.cn/oneinstack-letsencrypt.html)
 - [设置解析记录时提示冲突的原因](https://help.aliyun.com/knowledge_detail/39787.html)
 - [如何选购 SSL 证书](http://www.ituring.com.cn/article/273643)
