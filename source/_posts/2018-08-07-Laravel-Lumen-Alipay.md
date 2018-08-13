---
title: 基于Laravel/Lumen-支付宝支付(Alipay)
date: 2018-08-07 14:51:52
categories: Lumen
tags: [Laravel, Lumen]
---

> `latrell/Alipay`支付宝SDK在`Laravel5`的封装。该拓展包想要达到在`Laravel5/Lumen`框架下，便捷使用支付宝的目的。 by <u>[latrell/Alipay](https://github.com/latrell/Alipay)</u>
> 博客中还有另外一篇 [基于Lumen-支付宝支付(Payment)](http://blog.caoxl.com//2018/08/07/Lumen-Alipay/)

<!-- more -->

![支付宝调用流程](https://caoxl.com/images/alipay.png)

> 原图请看-->[手机网站支付快速接入](https://docs.open.alipay.com/203/105285/)

# 安装

```
    composer require latrell/alipay dev-master
```

本文封装中还使用了: [Payment](https://github.com/helei112g/payment)

```
    composer require "riverslei/payment:*"
```

本文的返回值使用了: [i18n](https://caoxl.com/2018/08/07/Lumen-il8n/)

# 使用

要使用支付宝SDK服务提供者，你必须自己注册服务提供者到`Laravel/Lumen`服务提供者列表中。 基本上有两种方法可以做到这一点。

## `Laravel`

找到 `config/app.php` 配置文件中，`ke`y为 `providers` 的数组，在数组中添加服务提供者。

```
    'providers' => [
        // ...
        'Latrell\Alipay\AlipayServiceProvider',
    ]
```

运行 `php artisan vendor:publish` 命令，发布配置文件到你的项目中。

## `Lumen`

在`bootstrap/app.php`里注册服务。

```
    // Register Service Providers
    $app->register(Latrell\Alipay\AlipayServiceProvider::class);
```

由于`Lumen`的`artisan`命令不支持`vendor:publish`,需要自己手动将`src/config`下的配置文件拷贝到项目的`config`目录下, 并将`config.php`改名成`latrell-alipay.php`, `mobile.php`改名成`latrell-alipay-mobile.php`, `web.php`改名成`latrell-alipay-web.php`.

- `latrell-alipay.sample.php`

```
    <?php
    return [
    
    	// 合作身份者id 以 `2088` 开头的16位纯数字
    	'partner_id' => '2088xxxxxxxxxxxxx',
    
    	// 卖家支付宝帐户
    	'seller_id' => 'xxxx'
    ];
```

- `latrell-alipay-mobile.sample.php`

```
    <?php
    return [
    
    	// 安全检验码，以数字和字母组成的32位字符
    	'key' => 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    
    	// 签名方式
    	'sign_type' => 'RSA',
    
    	// 商户私钥
    	'private_key_path' => __DIR__ . '/key/private_key.pem',
    
    	// 阿里公钥
    	'public_key_path' => __DIR__ . '/key/public_key.pem',
    
    	// 异步通知连接
    	'notify_url' => 'http://xxx'
    ];
```

- `latrell-alipay-web.sample.php`

```
    <?php
    return [
    
    	// 安全检验码，以数字和字母组成的32位字符
    	'key' => 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    
    	// 签名方式
    	'sign_type' => 'MD5',
    
    	// 服务器异步通知页面路径
    	'notify_url' => 'http://xxx',
    
    	// 页面跳转同步通知页面路径
    	'return_url' => 'http://xxx'
    ];
```

# 插件实例

## 支付申请

### 手机端

```
    // 创建支付单。
    $alipay = app('alipay.mobile');
    $alipay->setOutTradeNo('order_id');
    $alipay->setTotalFee('order_price');
    $alipay->setSubject('goods_name');
    $alipay->setBody('goods_description');

    // 返回签名后的支付参数给支付宝移动端的SDK。
    return $alipay->getPayPara();
```

### 网页

```
    // 创建支付单。
    $alipay = app('alipay.web');
    $alipay->setOutTradeNo('order_id');
    $alipay->setTotalFee('order_price');
    $alipay->setSubject('goods_name');
    $alipay->setBody('goods_description');
    
    $alipay->setQrPayMode('4'); //该设置为可选，添加该参数设置，支持二维码支付。

    // 跳转到支付页面。
    return redirect()->to($alipay->getPayLink());
```

# 项目应用

## 代码封装

> 支付是一个与钱打交道的操作, 必须确保交易正常.

- `app/Contract/PaymentMethod.php`

```
    <?php
    
    // Payment methods contract
    // @caoxl
    
    namespace App\Contract;
    
    interface PaymentMethod
    {
        // Validate payment parameters before pay action
        public function prepare(array &$params);
    
        // Pay action
        // @params Payment uses parameters
        // @return format:
        // [
        //    'err' => 0,
        //    'msg' => 'ok',
        //    'dat' => [...],  // optional
        // ]
        public function pay(array &$params): array;
    }
```

**所有支付类必须实现这个接口中定义的所有方法**

- `app/Http/Controllers/Payment/Alipay.php`

```
    <?php
    
    // Alipay operations
    // @caoxl
    
    namespace App\Http\Controllers\Payment;
    
    use
        Illuminate\Support\Facades\Validator,
        Illuminate\Http\Request,
        App\Traits\Tool,
        Payment\Common\PayException,
        Payment\Client\Refund,
        Payment\Config,
        App\Models\Finance\RefundLog;
    
    class Alipay implements \App\Contract\PaymentMethod
    {
        /**
         * @param array $params
         * @return array|bool
         */
        public function prepare(array &$params)
        {
            $validator = Validator::make($params, [
                'client'   => 'required|in:web,wap,mobile',
                'amount'   => 'require|numeric|min:0.01',
                'notify'   => 'required|url',
                'trade_no' => 'required',
                'desc'     => 'required',
            ]);
    
            if ($validator->fails()) {
                return [
                    'err' => 400,
                    'msg' => $validator->errors()->first(),
                ];
            }
    
            return true;
        }
    
        /**
         * @param array $params
         * @return array
         */
        public function pay(array &$params): array
        {
            if (true !== ($prepareRes = $this->prepare($params))) {
                return $prepareRes;
            }
    
            $amountEscape = in_array(env('APP_ENV'), ['local', 'test', 'stage'])
                ? 0.01
                : $params ['amount'];
    
            $alipay = app('alipay.' . $params['client']);
            $alipay->setOutTradeNo($params['trade_no']);
            $alipay->setTotalFee($amountEscape);
            $alipay->setSubject($params['desc']);
            $alipay->setNotifyUrl($params['notify']);
    
            if (isset($params['body'])
                && is_string($params['body'])
                && $params['body']
            ) {
                $alipay->setBody($params['body']);
            }
    
            $fillPayDatHandler = 'fillPayDataFor' . ucfirst($params['client']);
    
            if (! method_exists($this, $fillPayDatHandler)) {
                return [
                    'err' => 5001,
                    'msg' => Tool::sysMsg('MISSING_PAY_METHOD_HANDLER'),
                ];
            }
    
            return $this->$fillPayDatHandler($alipay, $params);
        }
    
        /**
         * @param $alipay
         * @param $params
         * @return array
         */
        protected function fillPayDataForMobile(&$alipay, $params): array
        {
            return [
                'err' => 0,
                'msg' => 'ok',
                'dat' => [
                    'params' => $alipay->getPayPara(),
                ],
            ];
        }
    
        /**
         * @param $alipay
         * @param $params
         * @return array
         */
        protected function fillPayDataForWeb(&$alipay, $params): array
        {
            $alipay->setAppPay('N');
    
            $validator = Validator::make($params, [
                'return' => 'required|url',
            ]);
    
            if ($validator->fails()) {
                return [
                    'err' => 400,
                    'msg' => $validator->errors()->first(),
                ];
            }
    
            // Enable QR pay, optional
            // See: <https://doc.open.alipay.com/support/hotProblemDetail.htm?spm=a219a.7386797.0.0.LjEOn6&source=search&id=226728>
            if (isset($params['qrpay'])
                && in_array($params['qrpay'], [0, 1, 2, 3, 4])
                && method_exists($alipay, 'setQrPayMode')
            ) {
                $alipay->setQrPayMode($params['qrpay']);
            }
    
            $alipay->setReturnUrl($params['return']);
    
            return [
                'err' => 0,
                'msg' => 'ok',
                'dat' => [
                    'url' => base64_encode($alipay->getPayLink()),
                ],
            ];
        }
    
        /**
         * @param $alipay
         * @param $params
         * @return array
         */
        protected function fillPayDataForWap(&$alipay, $params): array
        {
            return $this->fillPayDataForWeb($alipay, $params);
        }
    
        /**
         * @param $client
         * @return bool
         */
        public function tradeSuccess($client): bool
        {
            if (! in_array($client, ['wap', 'web', 'mobile'])) {
                return false;
            } elseif (! app('alipay.' . $client)->verify()) {
                return false;
            } elseif (! ($tradeStatus = ($_REQUEST['trade_status'] ?? false))
                || !in_array($tradeStatus, [
                    'TRADE_SUCCESS',
                    'TRADE_FINISHED'
                ])
            ) {
                return false;
            }
    
            return true;
        }
    
        public function payCallback($transHook): string
        {
            // TODO
        }
    
        protected function validate(array $params, array $rules)
        {
            $validator = Validator::make($params, $rules);
    
            if ($validator->fails()) {
                return [
                    'err' => 400,
                    'msg' => $validator->errors()->first(),
                ];
            }
    
            return true;
        }
    
        /**
         * @param array $params
         * @return array|bool
         */
        public function refund(array $params)
        {
            $createAt = time();
            $config   = config('custom')['alipay'] ?? [];
    
            if (true !== (
                $legalConfig = $this->validate($config, [
                    'app_id'          => 'required',
                    'sign_type'       => 'required|in:RSA,RSA2',
                    'use_sandbox'     => 'required',
                    'ali_public_key'  => 'required',
                    'rsa_private_key' => 'required'
                ]))) {
                return $legalConfig;
            } elseif (true !== ($legalParams = $this->validate($params, [
                    'paylog_id' => 'required|integer|min:1',
                    'id_type'   => 'required|in:trade_no,out_trade_no',
                    'trade_no'  => 'required',
                    'amount'    => 'required|numeric|min:0.01',
                    'operator'  => 'required',
                ]))) {
                return $legalParams;
            }
    
            try {
                $refundLog = RefundLog::wherePaylogId($params['paylog_id'])->first();
    
                $refundNo  = $refundLog
                    ? $refundLog->refund_no
                    : Tool::tradeNo(0, '04');
    
                $reason = $params['reason'] ?? Tool::sysMsg(
                        'REFUND_REASON_COMMON'
                );
    
                $data = [
                    'refund_fee' => $params['amount'],
                    'reason'     => $reason,
                    'refund_no'  => $refundNo,
                ];
    
                $data[$params['id_type']] = $params['trade_no'];
    
                $_data = [
                    'refund_no' => $refundNo,
                    'amount'    => $params['amount'],
                    'paylog_id' => $params['paylog_id'],
                    'operator'  => $params['operator'],
                    'reason_request' => $reason,
                ];
    
                $err = 0;
                $msg = 'ok';
    
                $ret = Refund::run(Config::ALI_REFUND, $config, $data);
    
                $processAt = time();
    
                if (isset($ret['code']) && ('10000' == $ret['code'])) {
                    $_data['process_at'] = date('Y-m-d H:i:s');
                    $_data['status'] = 1;
                } else {
                    $err = $ret['code'] ?? 5001;
                    $msg = $reasonFail = $ret['msg'] ?? Tool::sysMsg(
                            'REFUND_REQUEST_FAILED'
                    );
                }
    
                if (true !== ($updateOrInsertRefundLogRes = $this->updateOrInsertRefundLog(
                    $refundLog,
                    $_data,
                    $createAt
                ))) {
                    return $updateOrInsertRefundLogRes;
                }
    
                return [
                    'err' => $err,
                    'msg' => $msg,
                    'dat' => $ret,
                ];
    
            } catch (PayException $pe) {
                $status = ($refundLog && (1 == $refundLog->status)) ? 1 : 2;
                $_data['status']      = $status;
                $_data['reason_fail'] = $pe->getMessage();
                $_data['process_at']  = date('Y-m-d H:i:s');
    
                if (true !== ($updateOrInsertRefundLogRes = $this->updateOrInsertRefundLog(
                    $refundLog,
                    $_data,
                    $createAt
                ))) {
                    return $updateOrInsertRefundLogRes;
                }
    
                return [
                    'err' => '500X',
                    'msg' => $pe->getMessage(),
                ];
            } catch (\Exception $e) {
                return [
                    'err' => '503X',
                    'msg' => $e->getMessage(),
                ];
            }
        }
    
        /**
         * @param $refundLog
         * @param $data
         * @param $createAt
         * @return array|bool
         */
        protected function updateOrInsertRefundLog($refundLog, $data, $createAt)
        {
            if ($refundLog) {
                $processRes = RefundLog::whereId($refundLog->id)
                ->update();
            } else {
                date_default_timezone_set(env('APP_TIMEZONE', 'Asia/Shanghai'));
                $_data['create_at'] = date('Y-m-d H:i:s', $createAt);
                $processRes = RefundLog::insert($data);
            }
    
            return $processRes ? true : [
                $err = 5002,
                $msg = Tool::sysMsg('DATA_UPDATE_ERROR')
            ];
        }
    }
```

**代码说明:**

- `public function prepare(array &$params)`: 支付前验证
- `public function pay(array &$params): array`: 支付操作
- `protected function fillPayDataForMobile(&$alipay, $params): array`: 手机端支付宝
- `protected function fillPayDataForWeb(&$alipay, $params): array`: 网页端支付宝
- `protected function fillPayDataForWap(&$alipay, $params): array`: 电脑端支付宝
- `public function tradeSuccess($client): bool`: 验证交易
- `public function payCallback(array $params, array $rules)`: 支付后回调
- `public function refund(array $params)`: 支付宝退款
- `protect function updateOrInsertRefundLog($refundLog, $data, $createAt)`: 退款记录

# 附录
- 代码见: [Github SoulMate94/Lumen_Pay](https://github.com/SoulMate94/Lumen_Pay)

# 参考

- [latrell/Alipay](https://github.com/latrell/Alipay)
- [GitHub Laravel 扩展包 TOP 250](https://laravel-china.org/projects/filter/laravel-top250)
- [使用OpenSSL来产生RSA密钥](https://global.alipay.com/service/wap_cn/24)
- [App支付快速接入](https://docs.open.alipay.com/204/105297/)