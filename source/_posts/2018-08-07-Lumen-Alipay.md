---
title: 基于Lumen-支付宝支付(Payment)
date: 2018-08-07 10:12:31
categories: Lumen
tags: [Lumen, 支付宝, 支付]
---

> 基于Lumen-支付宝支付, 这里使用的是[Payment](https://github.com/helei112g/payment)第三方sdk by <u>[helei112g/payment](https://github.com/helei112g/payment)</u>

<!-- more -->

> **Payment是php版本的支付聚合第三方sdk**，集成了微信支付、支付宝支付、招商一网通支付。提供统一的调用接口，方便快速接入各种支付、查询、退款、转账能力。服务端接入支付功能，方便、快捷。
> > <u>[Payment使用文档](https://helei112g1.gitbooks.io/payment-sdk/content/)</u>
> > <u>[Payment使用常见问题汇总](https://helei112g1.gitbooks.io/payment-sdk/content/faq.html)</u>

# 环境准备

- `Payment` 需要 `PHP >= 5.6`以上的版本，并且同时需要PHP安装以下扩展
  - `cURL extension`
  - `mbstring`
  - `BC Math`
  - `Guzzle`
  > guzzle 是一个开源的php http请求lib, [项目地址](https://github.com/guzzle/guzzle)

## 安装`Payment`

```
    composer require "riverslei/payment:*"
```

## 先看几个`Payment Demo`

### 手机`app`支付`demo`

- `vendor/riverslei/example/ali/appCharge.php`

```php
    <?php
    
    require_once __DIR__ . '/../../autoload.php';
    
    use Payment\Common\PayException;
    use Payment\Client\Charge;
    use Payment\Config;
    
    date_default_timezone_set('Asia/Shanghai');
    $aliConfig = require_once __DIR__ . '/../aliconfig.php';
    
    // 订单信息
    $orderNo = time() . rand(1000, 9999);
    $payData = [
        'body'    => 'ali qr pay',
        'subject'    => '测试支付宝扫码支付',
        'order_no'    => $orderNo,
        'timeout_express' => time() + 600,// 表示必须 600s 内付款
        'amount'    => '0.01',// 单位为元 ,最小为0.01
        'return_param' => '123123',
        'client_ip' => isset($_SERVER['REMOTE_ADDR']) ? $_SERVER['REMOTE_ADDR'] : '127.0.0.1',// 客户地址
        'goods_type' => '1',
        'store_id' => '',
    ];
    
    try {
        $str = Charge::run(Config::ALI_CHANNEL_APP, $aliConfig, $payData);
    } catch (PayException $e) {
        echo $e->errorMessage();
        exit;
    }
    
    echo $str;// 这里如果直接输出到页面，&not 会被转义，请注意
```

### `wap`网站支付

- `vendor/riverslei/example/ali/wapCharge.php`

```php
    <?php

    require_once __DIR__ . '/../../autoload.php';
    
    use Payment\Common\PayException;
    use Payment\Client\Charge;
    use Payment\Config;
    
    date_default_timezone_set('Asia/Shanghai');
    $aliConfig = require_once __DIR__ . '/../aliconfig.php';
    
    // 订单信息
    $orderNo = time() . rand(1000, 9999);
    $payData = [
        'body'    => 'ali wap pay',
        'subject'    => '测试支付宝手机网站支付',
        'order_no'    => $orderNo,
        'timeout_express' => time() + 600,// 表示必须 600s 内付款
        'amount'    => '0.01',// 单位为元 ,最小为0.01
        'return_param' => 'tata',// 一定不要传入汉字，只能是 字母 数字组合
        'client_ip' => isset($_SERVER['REMOTE_ADDR']) ? $_SERVER['REMOTE_ADDR'] : '127.0.0.1',// 客户地址
        'goods_type' => '1',
        'store_id' => '',
    ];
    
    try {
        $url = Charge::run(Config::ALI_CHANNEL_WAP, $aliConfig, $payData);
    } catch (PayException $e) {
        echo $e->errorMessage();
        exit;
    }
    
    header('Location:' . $url);
```

### `web`电脑支付 即时到账接口

- `vendor/riverslei/example/ali/webCharge.php`

```php
    <?php
    
    require_once __DIR__ . '/../../autoload.php';
    
    use Payment\Common\PayException;
    use Payment\Client\Charge;
    use Payment\Config;
    
    date_default_timezone_set('Asia/Shanghai');
    $aliConfig = require_once __DIR__ . '/../aliconfig.php';
    
    // 订单信息
    $orderNo = time() . rand(1000, 9999);
    $payData = [
        'body'    => 'ali web pay',
        'subject'    => '测试支付宝电脑网站支付',
        'order_no'    => $orderNo,
        'timeout_express' => time() + 600,// 表示必须 600s 内付款
        'amount'    => '0.01',// 单位为元 ,最小为0.01
        'return_param' => '123123',
        'client_ip' => isset($_SERVER['REMOTE_ADDR']) ? $_SERVER['REMOTE_ADDR'] : '127.0.0.1',// 客户地址
        'goods_type' => '1',
        'store_id' => '',
    
        // 说明地址：https://doc.open.alipay.com/doc2/detail.htm?treeId=270&articleId=105901&docType=1
        // 建议什么也不填
        'qr_mod' => '',
    ];
    
    try {
        $url = Charge::run(Config::ALI_CHANNEL_WEB, $aliConfig, $payData);
    } catch (PayException $e) {
        echo $e->errorMessage();
        exit;
    }
    
    header('Location:' . $url);
```