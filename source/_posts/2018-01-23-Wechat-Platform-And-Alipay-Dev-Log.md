---
title: 支付宝/微信 「开发日志」
date: 2018-01-23 15:13:19
categories: ThirdParty
tags: [Wechat, 微信, Alipay, 支付宝, 第三方, ThirdParty]
---

调用第三方服务的时候总难一次性搞定的，下面把微信和支付宝服务集成到应用时遇到的“坑”简单总结下。

> 为啥要把支付宝和微信放到一起?? 因为他俩是基佬~~

<!--more-->

# 支付宝

- **官方 PHP SDK 中公私钥的换行方式会影响公私钥校验**
- **PHP 在支付宝开发后台填写公私钥时提示不匹配**

服务端语言是 `PHP`，通过支付宝提供的 `RSA` 秘钥工具生成的私钥（`应用私钥2048.txt`）格式是一行连续的，已取出头尾部的，未换行的字符串。而在上传应用公钥的时候，如果点击了“验证公钥正确性”，并使用弹出页面中提供的验签工具生成签名时候，需要私钥的是 **pkcs8**（JAVA 适用）格式，而为 PHP 生成的秘钥是 **pkcs1** 格式，因此需要转换。

**第一步：** 将非 JAVA 适用的私钥文件转换为标准 PEM 格式（64个字符串换一行）

```
    $raw_rsa_secret = file_get_contents('/path/to/应用私钥2048.txt');
    $secret_content = implode("\n", str_split($raw_rsa_secret, 64));
    $pem_key_pkcs1  = '-----BEGIN RSA PRIVATE KEY-----'.PHP_EOL.$secret_content.PHP_EOL.'-----END RSA PRIVATE KEY-----';
    file_put_contents('pkcs1.pem', $pem_key_pkcs1);
```

**第二步：** 将上一步转换后的标准 PEM 格式的 pkcs1 私钥 转换为 pkcs8 私钥

```
    openssl pkcs8 -topk8 -inform PEM -in pkcs1.pem -outform pem -nocrypt -out pkcs8.pem
```

**第三步：** 将上一部转换后的私钥`（pkcs8.pem）`复制到支付宝提供的验签工具对字符串 `a=123` 进行签名


**第四步：** 将签名结果复制的支付宝开发者后台进行确认公钥正确性

- **<u>https://b.alipay.com/order/serviceIndex.htm</u> 和 开放平台的支付宝公钥不一样。**

开放平台的接口就用开放平台的支付宝公钥，`mapi`网关的接口就用mapi网关的支付宝公钥。前者是新接口 ，后者是老接口使用。

- **notify_url 和 return_url 默认不能带自定义参数的**

即：`http://example.com/index.php?trade/payment/notify-alipay-app` 中 **? 后面的参数都会被支付宝忽略**。从而请求的是 `http://example.com/index.php`。

- **reurn_url 和 notify_url 的区别？**

对于 `notify_url` de说明详见参考的 服务器异步通知参数说明，因此这里只说下 `return_url`：

  - 交易完成后，一般情况支付宝会先通知 `notify_url`（POST方式），但是并不一定，因此**不要以先后顺序来做判断**，而要根据数据库的订单状态来判断。
  - 买家在支付成功后会看到一个支付宝提示交易成功的页面，该页面会停留几秒，然后会自动跳转回商户指定的同步通知页面，即 `GET return_url`。
  - 该方式仅仅在买家付款完成以后进行自动跳转，因此只会进行一次。
  - 该方式不是支付宝主动去调用商户页面，而是支付宝的程序利用页面自动跳转的函数，使用户的当前页面自动跳转。
  - 返回URL只有一分钟的有效期，超过一分钟该链接地址会失效，验证则会失败。
  
- **支付宝POST异步回调地址过来的数据，有的有 is_success 字段有的没有？**

移动支付接口，异步通知本来没有 `is_success`。手机网站支付接口，异步里也没有那个的参数的，但是同步是有的。


[ 移动支付的文档](https://docs.open.alipay.com/59/103665) && [快速查单](https://mbillexprod.alipay.com/enterprise/tradeOrder.htm)

> 程序执行完后必须打印输出“success”（不包含引号）。如果商户反馈给支付宝的字符不是success这7个字符，支付宝服务器会不断重发通知，直到超过24小时22分钟。
  一般情况下，25小时以内完成8次通知（通知的间隔频率一般是：4m,10m,10m,1h,2h,6h,15h）；
  
- **如何区分接口版本？**

  - 接口文档中网关是 openapi.alipay.com 的是 2.0 接口；
  - <u>[mapi（即时到账）](https://docs.open.alipay.com/62/103566/)</u> 是 1.0 接口；
  - <u>[手机网站支付](https://docs.open.alipay.com/60/103564)</u>是 1.0 接口。

  **不同 API 版本公私钥不通用。**


## 公私钥

参考这篇文档即可：<u>https://docs.open.alipay.com/291</u>。

## 支付

### app_pay

See: <u>https://docs.open.alipay.com/60/104790 </u>:

> app_pay=Y：尝试唤起支付宝客户端进行支付，若用户未安装支付宝，则继续使用wap收银台进行支付。商户若为APP，则需在APP的webview中增加alipays协议处理逻辑。

## 退款

### 即时到账批量退款有密接口

需要密码。可参考：<u>[即时到账批量退款有密接口](http://aopsdkdownload.cn-hangzhou.alipay-pub.aliyun-inc.com/demo/alipayrefund.zip)</u>

### 新版开放平台退款接口 `alipay.trade.refund`

*参考：* <u>[alipay.trade.refund(统一收单交易退款接口)](https://docs.open.alipay.com/api_1/alipay.trade.refund/)</u>。

**注意:**

`alipay.trade.refund` 的返回值中没有相关的ID来标识本次退款请求的。（微信有）


# 微信

  - **access_token => 区分 「网页授权」 和「微信公众号」**
  - **JSSDK 支付 => 注意中英输入法分号之差**
  
## 金额单位

微信支付时使用的金额单位一律是分，对于 PHP 这种弱类型语言来说尤其需要注意。

一般而言，应用服务器数据库里面的金额都是存的浮点型。因此在转换成整数的时候尤其需要小心。如果出现订单金额和实际支付金额不同的情况，在调用诸如原路退款接口的时候会出现“退款金额无效”的提示。

这个问题很早有人在 PHP 一次 <u>[‘bug’](https://bugs.php.net/bug.php?id=62385&edit=1)</u> 反馈中提到过，现象如下：

> Rasmus Lerdorf 的回答是：not a bug…

```
    echo intval(17.9 * 100);   // 期望 1790 而实际输出 1789
```

解决办法是：调用 `intval()` 之前使用 `round()` 先将浮点数四舍五入，或者先当作字符串处理一次后在调用 `intval()`。

```
    echo intval(round(17.9 * 100, 0));    // output: 1790
    
    // Or:
    function getIntFee(float $float) : int
    {
      $arr = explode('.', $float);
      
      if (! isset($arr[0])) {
          return intval($arr[0]);
      }
      
      return false;
    }
    
    $amount = getIntFee(17.9 * 100); // output: 1790
```

# 参考

- [支付宝开发文档](https://openhome.alipay.com/developmentDocument.htm)
- [支付宝 PHP SDK](https://docs.open.alipay.com/54/103419)
- [支付宝-App支付DEMO&SDK](https://docs.open.alipay.com/54/104509)
- [支付宝-生成RSA密钥-签名工具](https://docs.open.alipay.com/291/105971)
- [支付宝接口使用文档说明 支付宝异步通知(notify_url)与return_url](http://www.cnblogs.com/phirothing/archive/2013/01/20/2868917.html)
- [服务器异步通知参数说明](https://docs.open.alipay.com/59/103666)
- [蚂蚁金服-开放平台：账户中心-PID和公钥管理](https://openhome.alipay.com/platform/keyManage.htm)
- [蚂蚁金服-开放平台：历史接口接入指南](https://docs.open.alipay.com/58/103541/)
- Composer 支付组件：[helei112g/payment](https://github.com/helei112g/payment)、[latrell/alipay](https://github.com/Latrell/Alipay)