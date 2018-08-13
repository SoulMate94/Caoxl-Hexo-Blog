---
title: SSL延迟有多大？
date: 2018-02-06 14:47:27
categories: Internet
tags: [SSL]
---

据说，Netscape公司当年设计<u>[SSL协议](http://blog.caoxl.com/2018/02/06/SSL-TLS-Protocol-Operating-Mechanism/)</u>的时候，有人提过，将互联网所有链接都变成HTTPs开头的加密链接。

这个建议没有得到采纳，原因之一是HTTPs链接比不加密的HTTP链接慢很多。（另一个原因好像是，HTTPs链接默认不能缓存。）

自从我知道这个掌故以后，脑袋中就有一个观念：HTTPs链接很慢。但是，它到底有多慢，我并没有一个精确的概念。直到今天我从一篇[文章](http://www.semicomplete.com/blog/geekery/ssl-latency.html)中，学到了测量HTTPs链接耗时的方法。

<!--more-->

> 原文地址: http://www.ruanyifeng.com/blog/2014/09/ssl-latency.html

首先我解释一下，为什么HTTPs链接比较慢。

HTTPs链接和HTTP链接都建立在TCP协议之上。HTTP链接比较单纯，使用三个握手数据包建立连接之后，就可以发送内容数据了。

![](http://www.ruanyifeng.com/blogimg/asset/2014/bg2014092402.png)

上图中，客户端首先发送`SYN`数据包，然后服务器发送`SYN+ACK`数据包，最后客户端发送`ACK`数据包，接下来就可以发送内容了。这三个数据包的发送过程，叫做**TCP握手**。

再来看HTTPs链接，它也采用TCP协议发送数据，所以它也需要上面的这三步握手过程。而且，在这三步结束以后，它还有一个[SSL握手](http://blog.caoxl.com/2018/02/06/SSL-TLS-Protocol-Operating-Mechanism/)。

总结一下，就是下面这两个式子。

```
    HTTP耗时 = TCP握手
    
    HTTPs耗时 = TCP握手 + SSL握手
```

所以，`HTTPs`肯定比`HTTP`耗时，这就叫**SSL延迟**。

命令行工具[curl](http://blog.caoxl.com/2018/02/02/cURL-Notes/)有一个w参数，可以用来测量`TCP握手`和`SSL握手`的具体耗时，以访问支付宝为例。

```
    $ curl -w "TCP handshake: %{time_connect}, SSL handshake: %{time_appconnect}\n" -so /dev/null https://www.alipay.com
    
    TCP handshake: 0.022, SSL handshake: 0.064
```

上面命令中的w参数表示指定输出格式，`time_connect`变量表示TCP握手的耗时，`time_appconnect`变量表示**SSL握手**的耗时（更多变量请查看[文档](https://curl.haxx.se/docs/manpage.html)和[实例](https://blog.josephscott.org/2011/10/14/timing-details-with-curl/)），s参数和o参数用来关闭标准输出。

从运行结果可以看到，**SSL握手的耗时（64毫秒）大概是TCP握手（22毫秒）的三倍**。也就是说，在建立连接的阶段，HTTPs链接比HTTP链接要长3倍的时间，具体数字取决于CPU的快慢和网络状况。

所以，如果是对安全性要求不高的场合，为了提高网页性能，建议不要采用保密强度很高的数字证书。一般场合下，**1024位的证书已经足够了，2048位和4096位的证书将进一步延长SSL握手的耗时**。