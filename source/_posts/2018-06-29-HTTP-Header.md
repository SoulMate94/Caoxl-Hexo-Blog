---
title: HTTP 请求头/响应头
date: 2018-06-29 10:09:26
categories: Internet
tags: [Internet, HTTP, 请求头, 响应头]
---

> 学习**Web开发**不好好学习`HTTP`，将会“打拳不练功，到老一场空”，
你花在犯迷糊上的时间比你沉下心来学习`HTTP`的时间肯定会多很多。

<!-- more -->

# 常用请求头

- `Accept-Charset`:     用于指定客户端接受的字符集

- `Accept-Encoding`:    用于指定可接受的内容编码,如`Accept-Encoding:gzip.deflate`

- `Accept-Language`:    用于指定一种自然语言, 如`Accept-Language:zh-cn`

- `Host`:   用户指定被请求资源的`Internet`主机和端口号, 如:`Host:www.baidu.com`

- `User-Agent`:     客户端将它的操作系统, 浏览器和其他属性告诉服务器

- `Connection`:     当前连接是否保持, 如`Connection:Keep-Alive`

## 实例:

### `www.google.com`

- `Request Headers`

```
    :authority: www.google.com
    :method: GET
    :path: /
    :scheme: https
    accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
    accept-encoding: gzip, deflate, br
    accept-language: zh-CN,zh;q=0.9,en;q=0.8
    cache-control: max-age=0
    cookie: HSID=Ao92eQFZB7kOEw6yA; SSID=AlC3MG6I7bTxZDQO7; APISID=X9CY9NZ28EJTz_eV/AQWH2CR8lf0fNgW8e; SAPISID=rKRnvK8jf4WaTiP_/AzvzGc4aeAWEmw_Zy; SID=BgbU8VicgfUPXbPG_1K5CNKIObkJhzhMkCO0KwYGPAVkVZHtmeSjZ7H_DSPwqbW5bnqv2Q.; NID=133=bbY2KtALCHEfXVadHQT3yEU0qjiu-QrJXO8KRmKLBKvLzcbdGceq0jZVJQ8CG1FltiIWwr7qoDMKnYuJsNGPZ3RQCaPokGRdwatE_Pelzu9zzI5H-0NMP2KfSw-0k4G0SpY743QzHq4yd1XJEQU0iiKpwodX__rSOcuY-4Y8-7QnCZnHekLRSJxUj5GhQeGut-nWzsJq8sz6UHmD6GOmdJvt2EcrwahS47UdlOnm9A10acXIVEeziHbIcY_-DkiDmcQaNugksHQ; 1P_JAR=2018-6-29-2; SIDCC=AEfoLebf0c1-B3cWJBdIC56FZhmwxbqb-awIbH8dOJEdHA8pD_S71ZAbojm1tZwefxQ1mimuNz8
    upgrade-insecure-requests: 1
    user-agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36
    x-client-data: CI22yQEIpLbJAQjBtskBCI2aygEIqZ3KAQjXncoBCKijygE=
```

### `www.baidu.com`

- `Request Headers`

```
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
    Accept-Encoding: gzip, deflate, br
    Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
    Cache-Control: max-age=0
    Connection: keep-alive
    Cookie: BAIDUID=28C52FEE8FBF24DE95D26F820D40665B:FG=1; BIDUPSID=28C52FEE8FBF24DE95D26F820D40665B; PSTM=1522721732; __cfduid=d80a99d77873ca28f1bc17def9f921d161522722327; BDUSS=C00Y0x2ekZVYzdjcm5aT09QU2JsYXlJc3lhb2txU29jRzdvT1RLVFBRWmczQmhiQVFBQUFBJCQAAAAAAAAAAAEAAADV-TlJtbDM27XE1u648LTlt~IAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGBP8VpgT~FaM; MCITY=-257%3A; locale=zh; pgv_pvi=3210548224; BD_UPN=12314753; BD_HOME=1; H_PS_PSSID=26748_1430_21112_20697_26350_22159
    Host: www.baidu.com
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36
```

### 参数详解: `Requests`部分

> 格式: `Header` - 解释 - 示例

- `Accept` - 指定客户端能够接收的内容类型 - `Accept: text/plain, text/html`

- `Accept-Charset` - 浏览器可以接受的字符编码集。 - `Accept-Charset: iso-8859-5`

- `Accept-Encoding` - 指定浏览器可以支持的web服务器返回内容压缩编码类型。 - `Accept-Encoding: compress, gzip`

- `Accept-Language` - 浏览器可接受的语言 - `Accept-Language: en,zh`

- `Accept-Ranges` - 可以请求网页实体的一个或者多个子范围字段 - `Accept-Ranges: bytes`

- `Authorization` - `HTTP`授权的授权证书 - `Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`

- `Cache-Control` - 指定请求和响应遵循的缓存机制 - `Cache-Control: no-cache`

- `Connection` - 表示是否需要持久连接。（HTTP 1.1默认进行持久连接）- `Connection: close`

- `Cookie` - HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。 - `Cookie: $Version=1; Skin=new;`

- `Content-Length` - 请求的内容长度 - `Content-Length: 348`

- `Content-Type` - 请求的与实体对应的`MIME`信息 - `Content-Type: application/x-www-form-urlencoded`

- `Date` - 请求发送的日期和时间 - `Date: Tue, 15 Nov 2010 08:12:31 GMT`

- `Expect` - 请求的特定的服务器行为 - `Expect: 100-continue`

- `From` - 发出请求的用户的Email - `From: user@email.com`

- `Host` - 指定请求的服务器的域名和端口号 - `Host: www.zcmhi.com`

- `If-Match` - 只有请求内容与实体相匹配才有效 - `	If-Match: “737060cd8c284d8af7ad3082f209582d”`

- `If-Modified-Since` - 如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码 - `If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT`

- `If-None-Match` - 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 - `If-None-Match: “737060cd8c284d8af7ad3082f209582d”`

- `If-Range` - 如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为`Etag` - `If-Range: “737060cd8c284d8af7ad3082f209582d”`

- `If-Unmodified-Since` - 只在实体在指定时间之后未被修改才请求成功 - `If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT`

- `Max-Forwards` - 限制信息通过代理和网关传送的时间 - `Max-Forwards: 10`

- `Pragma` - 用来包含实现特定的指令 - `Pragma: no-cache`

- `Proxy-Authorization` - 连接到代理的授权证书 - `Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`

- `Range` - 只请求实体的一部分，指定范围 - `Range: bytes=500-999`

- `Referer` - 先前网页的地址，当前请求网页紧随其后,即来路 - `Referer: http://www.zcmhi.com/archives/71.html`

- `TE` - 客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息 - `TE: trailers,deflate;q=0.5`

- `Upgrade` - 向服务器指定某种传输协议以便服务器进行转换（如果支持） - `Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11`

- `User-Agent` - User-Agent的内容包含发出请求的用户信息 - `User-Agent: Mozilla/5.0 (Linux; X11)`

- `Via` - 通知中间网关或代理服务器地址，通信协议 - `Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)`

- `Warning` - 关于消息实体的警告信息 - `Warn: 199 Miscellaneous warning`
 
# 常见响应头

- `Server`:     使用的服务器名称, 如`Server:Apache/13.6(Unix)`

- `Content-Type`:   用来指明发送给接受者的实体正文的媒体类型. 如`Content-Type:text/html;charset=GBK`

- `Content-Encoding`:   与请求头`Accept-Encoding`相对应.告诉浏览器服务器采用的是什么压缩编码

- `Content-Language`:   描述了资源所用的自然语言,与`Accept-Language`相对应

- `Content-Length`:     指明实体正文的长度, 用以字节方式存储十进制数字来表示

- `Keep-Alive`:     保持连接的时间,如`Keep-Alive:timeout=5,max=120`

## 实例:

### `www.google.com`

- `Response Headers`

```
    alt-svc: quic=":443"; ma=2592000; v="43,42,41,39,35"
    cache-control: private, max-age=0
    content-encoding: br
    content-type: text/html; charset=UTF-8
    date: Fri, 29 Jun 2018 02:59:34 GMT
    expires: -1
    server: gws
    set-cookie: 1P_JAR=2018-06-29-02; expires=Sun, 29-Jul-2018 02:59:34 GMT; path=/; domain=.google.com
    set-cookie: SIDCC=AEfoLeZmeujSZGZUWN8cMFDTFhHJWI1_plOiVlNP-Zivyba_nD9nsZbftwsJJRyp3DxhNeULcwQ; expires=Thu, 27-Sep-2018 02:59:34 GMT; path=/; domain=.google.com; priority=high
    status: 200
    strict-transport-security: max-age=86400
    x-frame-options: SAMEORIGIN
    x-xss-protection: 1; mode=block
```

### `www.baidu.com`

- `Response Headers`

```
    Bdpagetype: 2
    Bdqid: 0x928027420000b0f6
    Cache-Control: private
    Connection: Keep-Alive
    Content-Encoding: gzip
    Content-Type: text/html;charset=utf-8
    Date: Fri, 29 Jun 2018 03:03:40 GMT
    Expires: Fri, 29 Jun 2018 03:03:40 GMT
    Server: BWS/1.1
    Set-Cookie: BDSVRTM=230; path=/
    Set-Cookie: BD_HOME=1; path=/
    Set-Cookie: H_PS_PSSID=26748_1430_21112_20697_26350_22159; path=/; domain=.baidu.com
    Strict-Transport-Security: max-age=172800
    Transfer-Encoding: chunked
    X-Ua-Compatible: IE=Edge,chrome=1
```

### 参数详解: `Responses`部分

> 格式: `Header` - 解释 - 示例

- `Accept-Ranges` - 表明服务器是否支持指定范围请求及哪种类型的分段请求 - `Accept-Ranges: bytes`

- `Age` - 	从原始服务器到代理缓存形成的估算时间（以秒计，非负） - `Age: 12`

- `Allow` - 对某网络资源的有效的请求行为，不允许则返回405 - `Allow: GET, HEAD`

- `Cache-Control` - 告诉所有的缓存机制是否可以缓存及哪种类型 - `Cache-Control: no-cache`

- `Content-Encoding` - web服务器支持的返回内容压缩编码类型。 - `Content-Encoding: gzip`

- `Content-Language` - 响应体的语言 - `Content-Language: en,zh`

- `Content-Length` - 响应体的长度 - `Content-Length: 348`

- `Content-Location	` - 请求资源可替代的备用的另一地址 - `请求资源可替代的备用的另一地址`

- `Content-MD5` - 返回资源的MD5校验值 - `Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==`

- `Content-Range` - 在整个返回体中本部分的字节位置 - `Content-Range: bytes 21010-47021/47022`

- `Content-Type` - 返回内容的MIME类型 - `Content-Type: text/html; charset=utf-8`

- `Date` - 	原始服务器消息发出的时间 - `Date: Tue, 15 Nov 2010 08:12:31 GMT`

- `ETag` - 请求变量的实体标签的当前值 - `ETag: “737060cd8c284d8af7ad3082f209582d”`

- `Expires` - 响应过期的日期和时间 - `Expires: Thu, 01 Dec 2010 16:00:00 GMT`

- `Last-Modified` - 请求资源的最后修改时间 - `Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT`

- `Location` - 用来重定向接收方到非请求URL的位置来完成请求或标识新的资源 - `Location: http://www.zcmhi.com/archives/94.html`

- `Pragma` - 包括实现特定的指令，它可应用到响应链上的任何接收方 - `Pragma: no-cache`

- `Proxy-Authenticate` - 它指出认证方案和可应用到代理的该URL上的参数 - `Proxy-Authenticate: Basic`

- `refresh` - 应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持） - `Refresh: 5; url=http://www.zcmhi.com/archives/94.html`

- `Retry-After` - 如果实体暂时不可取，通知客户端在指定时间之后再次尝试 - `Retry-After: 120`

- `Server` - web服务器软件名称 - `Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)`

- `Set-Cookie` - 设置Http Cookie - `	Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1`

- `Trailer` - 指出头域在分块传输编码的尾部存在 - `Trailer: Max-Forwards`

- `Transfer-Encoding` - 文件传输编码 - `Transfer-Encoding:chunked`

- `Vary` - 告诉下游代理是使用缓存响应还是从原始服务器请求 - `Vary: *`

- `Via` - 告知代理客户端响应是通过哪里发送的 - `Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)`

- `Warning` - 警告实体可能存在的问题 - `Warning: 199 Miscellaneous warning`

- `WWW-Authenticate` - 表明客户端请求实体应该使用的授权方案 - `WWW-Authenticate: Basic`

# `HTTP Header`可选字段

- `Cache-Control/Pragma`
  - `Public`:   所有的内容都将被缓存, 在响应头中设置
  - `Private`:  内容只缓存到私有缓存中, 在响应头中设置
  - `no-cache`: 所有内容都不会被缓存, 在请求头和响应头中设置
  - `no-store`: 所有内容都不会被缓存到缓存或`Internet`临时文件中, 在响应头中设置
  - `must-revalidation/proxy-revalidation`: 如果缓存内容失效, 请求必须发送到服务器/代理以进行重新验证,在请求头中设置
  - `max-age=xxx`:  缓存的内容将在`xxx`秒后失效, 这个选项只在`Http 1.1`中可用, 和`Last-Modified`一起使用时优先级较高,在响应头中设置
  
  
# `PHP`设置`HTTP`请求头/响应头

## 使用`header`函数

- 设置请求头

```
    header('Accept-Language:zh-cn');
```

- 设置响应头

```
    header('Server:Apache/13.6(Unix)');
```

> 多个直接要写多个`header`,不可以连接在一起!


## 使用`fsockopen`函数

- `1.php`

```php
    <?php
    
    $fp = fsockopen("test.com", 80,$errNo,$errStr, 30);
    
    if (!$fp) {
        echo "$errStr ($errNo)",PHP_EOL;
    } else {
        $out  = "GET /2.php HTTP/1.1 \r\n";
        $out .= "Host: test.com \r\n";
        $out .= "name:cxl \r\n";
        $out .= "Connection: Close \r\n\r\n";
    
        fwrite($fp, $out);
    
        while (!feof($fp)) {
            echo fgets($fp, 128);
        }
    
        fclose($fp);
    }
```

- `2.php`

```php
    print_r(getallheaders())
```

会返回自己设置请求的头信息

## 使用`cURL`组件的使用

# `PHP`获取请求头/响应头

- `getallheaders` - 获取所有`HTTP`请求标头, (或`$_SERVER`)
- `headers_list`  - 返回已发送(或准备发送)的响应头列表

# 参考

- [HTTP Header 详解](https://kb.cnblogs.com/page/92320/)
- [14 Header Field Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)