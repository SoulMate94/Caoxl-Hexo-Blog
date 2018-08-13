---
title: JWT 基础知识
date: 2018-01-17 10:11:35
categories: PHP
tags: [JWT, PHP]
---

## 基于Token的身份验证——JWT

> 基于Token的身份验证用来替代传统的cookie+session身份验证方法中的session。

<!--more-->

## JWT是啥？

JWT就是一个字符串，经过加密处理与校验处理的字符串，形式为：

```
    A.B.C
    header.payload.secret
```
 - A由JWT头部信息`header`加密得到
 - B由JWT用到的身份验证信息`json`数据加密得到
 - C由A和B加密得到，是校验部分



## 怎样生成A？ (header)

header格式为：

```
    {
        "typ": "JWT",
        "alg": "HS256" 
    }
```

它就是一个json串，两个字段是必须的，不能多也不能少。`alg`字段指定了生成C的算法，默认值是 `HS256`
将header用base64加密，得到A
通常，JWT库中，可以把A部分固定写死，用户最多指定一个 `alg` 的取值


## 怎样计算B？ (payload)

根据JWT claim set[用base64]加密得到的。claim set是一个json数据，是表明用户身份的数据，可自行指定字段很灵活，也有固定字段表示特定含义（但不一定要包含特定字段，只是推荐）。
这里偷懒，直接用php中的代码来表示claim set了，重在说明字段含义:

```
    $token = array(
        "iss" => "http://example.org",   #非必须。issuer 请求实体，可以是发起请求的用户的信息，也可是jwt的签发者。
        "iat" => 1356999524,                #非必须。issued at。 token创建时间，unix时间戳格式
        "exp" => "1548333419",            #非必须。expire 指定token的生命周期。unix时间戳格式
        "aud" => "http://example.com",   #非必须。接收该JWT的一方。
        "sub" => "jrocket@example.com",  #非必须。该JWT所面向的用户
        "nbf" => 1357000000,   # 非必须。not before。如果当前时间在nbf里的时间之前，则Token不被接受；一般都会留一些余地，比如几分钟。
        "jti" => '222we',     # 非必须。JWT ID。针对当前token的唯一标识
    
        "GivenName" => "Jonny", # 自定义字段
        "Surname" => "Rocket",  # 自定义字段
        "Email" => "jrocket@example.com", # 自定义字段
        "Role" => ["Manager", "Project Administrator"] # 自定义字段
    );
```

JWT遵循 `RFC7519`，里面提到claim set的json数据中，自定义字段的key是一个string，value是一个json数据。因此随意编写吧，很灵活。

个人初学，认为一个最基本最简单最常用的claim set为：

```
    $token=array(
        "user_id" => 123456,     #用户id，表明用户
        "iat"     => 1356999524, #token发布时间
        "exp"     => 1556999524, #token过期时间
    );
```

将claim set加密后得到`B`，学名`payload`

## 怎样计算C？ (secret)

将`A.B`使用HS256加密（其实是用header中指定的算法），当然加密过程中还需要密钥（自行指定的一个字符串）。
加密得到 `C` ，学名`signature`，其实就是一个字符串。作用类似于CRC校验，保证加密没有问题。

好了，现在`A.B.C`就是生成的token了。

## 怎样使用token？

可以放到HTTP请求的请求头中，通常是`Authorization`字段。
也有人说放到cookie。不过移动端app用cookie似乎不方便。


## token应用流程？
 1.初次登录：用户初次登录，输入用户名密码
 2.密码验证：服务器从数据库取出用户名和密码进行验证
 3.生成JWT：服务器端验证通过，根据从数据库返回的信息，以及预设规则，生成JWT
 4.返还JWT：服务器的HTTP RESPONSE中将JWT返还
 5.带JWT的请求：以后客户端发起请求，HTTP REQUEST HEADER中的Authorizatio字段都要有值，为JWT
 
 