---
title: 各种操作系统下清空DNS缓存
date: 2018-07-26 15:39:50
categories: Internet
tags: [Internet, DNS]
---

# `Windows`

- 清空`DNS`缓存

```
    ipconfig /flushdns
```

<!-- more -->

- 查看`DNS`缓存

```
    ipconfig /displaydns
```

# `Mac OSX`下

```
    lookupd -flushcache
    
    # 或
    
```

# `Linux`下

> 在`Linux`中, `nscd`进程负责管理`DNS`缓存
> 要清空`DNS`缓存, 重启`nscd`守护进程就行了

- 清除 `nscd dns` 缓存

```
    service nscd restart
    
    # 或
    sudo /etc/init.d/nscd restart
    
    # 或
    service nscd reload
```

- 清除 `dnsmasq dns` 缓存

```
    sudo /etc/init.d/dnsmasq restart
    
    # 或
    service dnsmasq restart
```

- 清除`BIND`缓存服务器的`dns`缓存

```
    /etc/init.d/named restart
    
    # 或
    rndc restart
    
    # 或
    rndc exec
```