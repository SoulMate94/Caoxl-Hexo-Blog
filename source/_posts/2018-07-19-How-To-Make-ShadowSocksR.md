---
title: 在服务器上搭建 「ShadowSocksR」
date: 2018-07-19 14:22:07
categories: Linux
tags: Linux
---

> 墙外的世界很精彩, 墙外的世界很美好, 墙外的世界你值得拥有

<!-- more -->

# 服务器上安装`SSR`

- 1. 下载

```
    wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
```

- 2. 赋予可执行权限

```
    chmod +x shadowsocksR.sh
```

- 3. 运行

```
    ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

- 4. 出现以下内容, 输入密码和端口号(自定义)

```
    [root@caoxianliang ssr]# ./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
    
    #############################################################
    # One click Install ShadowsocksR Server                     #
    # Intro: https://shadowsocks.be/9.html                      #
    # Author: Teddysun <i@teddysun.com>                         #
    # Github: https://github.com/shadowsocksr/shadowsocksr      #
    #############################################################
```

- 5. 安装完成, 以下会输出在服务器上, 保存.

```
    Starting ShadowsocksR success
    
    Congratulations, ShadowsocksR server install completed!
    Your Server IP        :  *** 
    Your Server Port      :  *** 
    Your Password         :  *** 
    Your Protocol         :  origin 
    Your obfs             :  plain 
    Your Encryption Method:  aes-256-cfb 
    
    Welcome to visit:https://shadowsocks.be/9.html
    Enjoy it!

```

- 6. 在`阿里云-云服务器ECS-网络和安全组-安全组配置`中开放你设置的端口号


# `SSR`客户端

```
    链接：https://pan.baidu.com/s/18FOGMNwcd4trWQrxQgN-ZA 密码：oywm
```

以上的链接也许会过期, 过期就自己去网上找吧~~

下载完成后, 将服务器返回的配置, 输入到客户端即可.


# 参考

- [来自一个鸡的博客: SSR一键安装](https://blog.lilis.xin/2018/07/19/one-key-installation-ssr/)
- [Shadowsocks非官方网站](https://shadowsocks.be/9.html)