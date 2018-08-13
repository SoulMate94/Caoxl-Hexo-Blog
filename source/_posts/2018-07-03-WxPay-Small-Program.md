---
title: 微信小程序 「微信支付」
date: 2018-07-03 13:53:45
categories: ThirdParty
tags: [微信小程序, 微信支付, ThirdParty]
---

> 新项目是一个小程序,需要使用到支付~

<!-- more -->

# 业务流程

![小程序支付](https://caoxl.com/images/wxpay.jpg)

商户系统和微信支付系统主要交互:

- 1. 小程序内调用登录接口, 获取到用户的`openid`; API参见[小程序登录API](https://developers.weixin.qq.com/miniprogram/dev/api/api-login.html?t=20161122)
- 2. 商户`server`调用支付统一下单; API参见[统一下单API](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=9_1&index=1)
- 3. 商户`server`调用再次签名; API参见[再次签名](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_7&index=3)
- 4. 商户`server`接收支付通知; API参见[支付结果通知API](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=9_7)
- 5. 商户`server`查询支付结果; API参见[查询订单API](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=9_2)


# 直接上源码

## 前端部分

> 调用`wx.requestPayment(OBJECT)`发起微信支付

```
    wx.requestPayment({
        'timeStamp': '',
        'nonceStr': '',
        'package': '',
        'signType': 'MD5',
        'paySign': '',
        'success': function (res) {},
        'fail': function (res) {},
        'complete': function (res) {}
    });
```

**参数说明:**

- `timeStamp`:  时间戳, 即当前的时间
- `nonceStr`:   随机字符串, 长度为32个字符以下
- `package`:    统一下单接口返回的`prepay_id`参数值, 提交格式如: `prepay_id=*`
- `signType`:   签名类型, 默认为`MD5`, 支持`HMAC-SHA256`和`MD5`, 注意此处需要与统一下单的签名类型一致
- `paySign`:    签名, 详见:[微信公众号支付帮助文档](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=4_3)
- `success`:    接口调用成的回调函数
- `fail`:       接口调用失败的回调函数
- `complete`:   接口调用结束的回调函数(调用成功、失败都会执行)


**数据签名举例**

```
    paySign = MD5(appId=wxd678efh567hg6787&nonceStr=5K8264ILTKCH16CQ2502SI8ZNMTM67VS&package=prepay_id=wx2017033010242291fcfe0db70013231072&signType=MD5&timeStamp=1490840662&key=qazwsxedcrfvtgbyhnujmikolp111111) = 22D9B4E54AB1950F51E0649E8810ACD6
```

## 后端部分

### 发起支付

- `pay`

```php
    public function pay(Session $ssn, Request $req)
        {
            $user = $ssn->get('user');
    
            $_params = [
                'appid'      => 'your_appid',
                'mch_id'     => '1439831202',
                'nonce_str'  => md5(microtime()),
                'body'       => 'JSAPI支持测试',
                'attach'     => '支持测试',
                'fee_type'   => 'CNY',
                'total_fee'  => $req->all()['total_fee'],
                'notify_url' => 'https://skb-api.sciclean.cn/test/pay/payCallback',
                'trade_type' => 'JSAPI',
                'openid'     => $user['openid'],
                'out_trade_no'     => md5(time()),
                'spbill_create_ip' => $this->getRealIP()
            ];
    
            $_sign = '';
            $WxPay = '<xml>';
            ksort($_params);
    
            foreach ($_params as $k => $v) {
                if(is_scalar($v) && 'sign' != $k && '' != $v) {
                    $_sign .= $k . '=' . $v . '&';
                    $WxPay .= '<'.$k.'>'.$v.'</'.$k.'>';
                }
            }
    
            $sign  = $this->getSign($_sign);
            $WxPay .= '<sign>'.$sign.'</sign></xml>';
            $url   = 'https://api.mch.weixin.qq.com/pay/unifiedorder';
            $res   = $this->curl_post_ssl($url, $WxPay);
            $res   = $this->xmlToArray($res);
    
            $params = [
                'appId'     => $res['appid'],
                'nonceStr'  => $res['nonce_str'],
                'package'   => 'prepay_id='.$res['prepay_id'],
                'signType'  => 'MD5',
                'timeStamp' => ''.time()
            ];
    
            ksort($params);
    
            $sign = '';
            foreach($params as $k => $v) {
                $sign .= $k . '=' . $v . "&";
            }
            $params['paySign'] = $this->getSign($sign);
    
            return Tool::jsonResp([
                'err' => 0,
                'msg' => 'Order Create Success!',
                'dat' => $params
            ]);
        }
```

**参数说明:**

- `appid` - 小程序ID
- `mch_id` - 商户号
- `device_info` - 设备号
- `nonce_str` - 随机字符串
- `sign` - 签名
- `sign_type` - 签名类型
- `body` - 商品描述
- `detail` - 商品详情
- `attach` - 附加数据
- `out_trade_no` - 商户订单号
- `fee_type` - 标价币种
- `total_fee` - 标价金额
- `spbill_create_ip` - 终端IP
- `notify_url` - 通知地址
- `openid` - 用户标识
- `trade_type` - 交易类型


### 申请退款

- `refund`

```php
    public function refund(Session $ssn)
    {
        $user = $ssn->get('user');

        $_params = [
            'appid'       => 'your_appid',
            'mch_id'      => '1439831202',
            'nonce_str'   => md5(microtime()),
            'fee_type'    => 'CNY',
            'total_fee'   => 100,
            'refund_fee'  => 100,
            'notify_url'  => 'https://skb-api.sciclean.cn/test/pay/payCallback',
            'refund_desc' => 'JSAPI退款测试',
            'transaction_id'   => '4200000153201806292573121489',
            //'out_trade_no'     => '35f3751220e4c1d7396e8b1bc4d05ee41',
            'out_refund_no'    => md5(time())
        ];

        $sign  = '';
        $WxPay = '<xml>';

        ksort($_params);

        foreach ($_params as $k => $v) {
            if("sign" != $k && !is_null($v)) {
                $sign  .= $k . "=" . $v . "&";
                $WxPay .= '<'.$k.'>'.$v.'</'.$k.'>';
            }
        }

        $sign  = $this->getSign($sign);
        $WxPay .= '<sign>'.$sign.'</sign></xml>';

        $url   = 'https://api.mch.weixin.qq.com/pay/unifiedorder';
        $res   = $this->curl_post_ssl($url, $WxPay);

        if ($res) {
            $res = Tool::xmlToArray($res);

            return Tool::jsonResp([
                'err' => 0,
                'msg' => 'Refund Success!',
                'dat' => $res
            ]);
        }

        return Tool::jsonResp([
            'err' => -1,
            'msg' => 'Refund Failed!',
            'dat' => []
        ]);
    }
```

**参数说明:**

- `appid` - 小程序ID
- `mch_id` - 商户号
- `nonce_str` - 随机字符串
- `sign` - 签名
- `sign_type` - 签名类型
- `transaction_id` - 微信订单号
- `out_trade_no` - 商户订单号 **与微信订单号 二选一即可**
- `out_refund_no` - 商户退款单号
- `total_fee` - 订单金额
- `refund_fee` - 退款金额
- `refund_fee_type` - 货币种类
- `refund_desc` - 退款原因
- `refund_account` - 退款资金来源
- `notify_url` - 退款结果通知url


**注意:**
1. 交易时间超过一年的订单无法提交退款
2. 微信支付退款支持单笔交易分多次退款，多次退款需要提交原支付订单的商户订单号和设置不同的退款单号。
申请退款总金额不能超过订单金额。 **一笔退款失败后重新提交，请不要更换退款单号，请使用原商户退款单号**
3. 请求频率限制：`150qps`，即每秒钟正常的申请退款请求次数不超过`150`次
错误或无效请求频率限制：`6qps`，即每秒钟异常或错误的退款申请请求不超过`6`次
4. 每个支付订单的部分退款次数不能超过`50`次

### 用到的方法

- `getRealIP`

```
    protected function getRealIP()
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
```

- `xmlToArray`

```
    protected function xmlToArray(string $xml)
    {
        return json_decode(json_encode(simplexml_load_string(
            $xml,
            'SimpleXMLElement',
            LIBXML_NOCDATA
        )), true);
    }
```

- `arrayToXML`

```
    protected function arrayToXML(array $array, string &$xml): string
    {
        foreach ($array as $key => &$val) {
            if (is_array($val)) {
                $_xml = '';
                $val  = self::arrayToXML($val, $_xml);
            }

            $xml .= "<key>$val</key>";
        }

        unset($val);

        return $xml;
    }
```

- `jsonResp`

```
    private function jsonResp(
        $data,
        int $status = 200,
        bool $unicode = true
    ) {
        $unicode = $unicode ? JSON_UNESCAPED_UNICODE : null;

        $data = json_encode($data, $unicode);

        return response($data)
               ->header('Content-Type', 'application/json; charset=utf-8');
    }
```

- `nonceStr`

```
    protected function nonceStr(): string
    {
        return mt_rand(time(), time()+rand());
    }
```

- `sign`

```
    protected function sign(array $params, string $key)
    {
        $sign = '';
        ksort($params);
        foreach ($params as $k => $v) {
            if (is_scalar($v) && ('' != $v) && ('sign' != $k)) {
                $sign .= $k.'='.$v.'&';
            }
        }
        $sign .= '$key='.$key;
        $sign  = strtoupper(md5($sign));

        return $sign;
    }
```


# 参考

- [小程序调起支付API](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_7&index=5)
- [统一下单](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=9_1)
- [申请退款](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=9_4)

