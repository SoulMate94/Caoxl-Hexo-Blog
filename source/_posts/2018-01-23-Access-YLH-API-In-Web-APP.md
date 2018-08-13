---
title: 接入云联惠 API「开发日志」
date: 2018-01-23 16:21:14
categories: ThirdParty
tags: [Web, API, 云联惠, 第三方]
---

云联惠授权登录完全就是模仿微信来的，比较简单，简述如下。

<!--more-->

# 基本过程

- **获取 code**

首先，需要引导用户去云联惠提供的 `oauth2` 登录入口，地址为：

```
    https://opentest.yunlianhui.cn/token/authorize?client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&state=STATE&response_type=code&scope=SCOPE
```

如果用户在这个入口登录并授权之后，将会跳转到 `REDIRECT_URI`，并带上 `GET` 参数 `code=XXXX&state=test`。

`REDIRECT_URI` 是我们的应用服务器提供的 `URL`，因此需要保存这个 `code` 值，用于下一步操作。

- **用 code 换 access_token（令牌）**

```
    curl --request POST \
    --url https://opentest.yunlianhui.cn/token/authorize/accesstoken \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/x-www-form-urlencoded' \
    --data 'code=CODE&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&grant_type=authorization_code&redirect_uri=REDIRECT_URI

```

注意，这里的 `code` 失效时间很短，因此需要尽快去换取 `access_token`。

- **用 access_token 获得用户基本信息/账户等信息**

这里能获取到什么数据，首先是根据第一步获取 `code` 时的 `scope` 参数指定的。可以用 `+` 获得多种 `scope` 的信息，比如：`scopte=basic_info+account_info`。

其次，还有实际调用的接口来决定的，比如基本信息调用的是 `/api/basic_info` 接口，账户信息调用的是 `/api/account_info` 接口。

**格式如下：**

```
    curl -X POST \
    --url https://opentest.yunlianhui.cn/api/basic_info \
    --header 'cache-control: no-cache' \
    --header 'content-type: application/x-www-form-urlencoded' \
    --data 'client_id=CLIENT_ID&access_token=ACCESS_TOKEN&timestamp=YYYY-MM-DD HH:ii:ss&sign=SIGN_STRING'
```

或者通过 PHP 使用 cURL：

```
    function post(string $uri, array $headers = [], array $params = [])
    {
      $ch = curl_init();
      curl_setopt_array($ch, [
        CURLOPT_URL => $uri,
        CURLOPT_HEADER => $headers,
      ]);
      $res = curl_exec($ch);
      curl_close($ch);
      
      return $res;
    }
```

其中，这里的 `sign` 只需要 `md5` + `转大写` 就行了。如果成功则返回值如下：

```
    {
      "member_id":"FF1",
      "mobile":"15669714243",
      "rcm_id":"DD1",
      "email":"69524839@163.com",
      "error_code":0
    }
```

# FAQ

- **如何获得 client_id, appkey？**

设置一个回调地址（redirect_uri），技术人员给你：client_id，client_secret。

- **什么时候用 test 什么时候用正式？**

完成开发之后提交审核通过用正式。

- **完成开发之后提交审核通过用正式。**

  - `basic_info：` 访问会员的基本信息，时效性为 1 个月。
  - `account_info：` 访问会员的账户信息，时效性为 1 个月。
  - `points：` 产生积分返还将会扣除会员的库存积分，时效性为 1 个月。

- **签名规则？**

  - 将参数列表按名称的 ASCII 顺序排序
  - 然后把所有参数按顺序拼接：`参数1名参数1值``参数2名参数2值[…参数n名参数n值]`
  - 在拼接得到的字符串前后再拼接上 `client_secret`
  - 将上面的结果进行一次 `md5` 计算，最后全部`转为大写`
  
  用 PHP 表示就是：
  
```
    $params = [
      'p1' => 'v1',
      'p2' => 'v2',
      'p3' => 'v3',
    ];
    
    $client_secret = 'secret';
    
    ksort($params);
    
    $strToBeSigned = $client_secret;
    
    foreach ($params as $pName => $pVal) {
      $strToBeSigned .= $pName.$pVal;
    }
    
    $strToBeSigned .= $client_secret;
    
    $sign = strtouppler(md5($strToBeSigned));

```


# 参考

- <u>[MD5-wikipedia](https://zh.wikipedia.org/wiki/MD5)</u>
- RFC：<u>[RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)</u> && <u>[OAuth2 RFC6749中文翻译](http://colobu.com/2017/04/28/oauth2-rfc6749/)</u>
