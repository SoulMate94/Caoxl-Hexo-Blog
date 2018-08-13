---
title: 快钱支付 「开发日志」
date: 2018-01-23 15:48:24
categories: ThirdParty
tags: [快钱, 支付, 第三方, ThirdParty]
---

在二次开发江湖外卖中集成快钱支付

简单记录下步骤。

<!--more-->

# 说明

## 开发相关

- 明确网关版本是移动端还是 PC 端

两者的 demo 和文档不同，为了集成时少遇到点问题，必须明确当时是在接入什么端，然后再去看对应的文档和demo。

- 签名

商户提交给快钱的参数需要被签名，使用商户的私钥；快钱返回给商户的参数需要被验签，使用快钱的公钥。

- 二次封装

快钱提供的 demo（PHP）写的很随意和简陋，也不规范，建议自己封装好再放入系统使用。

## 公钥配置

需要登录快钱中心，上传商家的公钥，和下载快钱的公钥。

然后在项目中，**使用商家的私钥来加密支付交易数据，使用快钱的公钥来解密回调数据。**

对于生成商家公私钥，可以参考文档，使用快钱提供的工具来生成，这里就不细说了。

## 后台管理

登录地址为 <u>https://mrs.99bill.com</u>，要登录需要先在系统／浏览器安装一个快钱的证书。

# 支付接口开发

江湖外卖默认提供的支付方式是在后台设置中自行安装的，我在接入快钱的时候也想做成这样，因此在了解了江湖外卖支付插件开发的规范后，也实现了在后台可以自行安装块钱支付的功能。

> 没有文档，debug 源码例子后照葫芦画瓢来的。

首先，江湖外卖支付插件都是放在 `system/plugins/payments `目录下的。可以看到默认包含了：`alipay/money/paypay/stripe/wxpay`。

因此，我在这个目录下创建了一个 `kuaiqian` 的子目录。里面只有两个文件：

- **config.php**

这个配置文件每个支付方式都有，是用来在后台读取生成 HTML 的。内容如下：

```
    <?php
    
    return [
        'code' => 'kuaiqian',
        'name' => '快钱支付',
        'content' => '中国领先的互联网金融平台',
        'website' => 'https://www.99bill.com/',
        'version'   => '0.1',
        'currency'  => '人民币',
        'config' => [
            'kq_merchant_id' => [
                'text'  => '人民币网关账号',
                'desc'	=> '该账号为11位人民币网关商户编号+01,该参数必填',
                'type'  => 'text',
            ],
            'kq_gateway_mobile' => [
                'text'  => '快钱移动网关',
                'desc'	=> '<span style="color:red;">*</span>和PC网关必填其一(默认)',
                'type'  => 'text',
            ],
            'kq_gateway_pc' => [
                'text'  => '快钱PC网关',
                'desc'	=> '<span style="color:red;">*</span>和移动网关必填其一',
                'type'  => 'text',
            ],
            'kq_page_url' => [
                'text'  => '接收支付结果的页面地址',
                'desc'	=> '该参数一般置为空即可',
                'type'  => 'text',
            ],
            'kq_bg_url' => [
                'text'  => '服务器接收支付结果的后台地址',
                'desc'	=> '该参数务必填写，不能为空',
                'type'  => 'text',
            ],
        ],
    ];
```

其中对于使用来说，**最主要的就是「人民币网关账号」（配置时注意要加上 01）**。最后两个接受地址可以不用配置，我在代码中写好了。

- **kuaiqian.php**

这就是快钱支付接口的主要文件了。

```
   <?php
   // 注意类命名格式必须要这种
   class Payment_Kuaiqian
   {
       private $code = 'kuaiqian';
       private $gatewaySandboxPC = 'https://sandbox.99bill.com/gateway/recvMerchantInfoAction.htm';
       private $gatewaySandboxMobile = 'https://sandbox.99bill.com/mobilegateway/recvMerchantInfoAction.htm';
       private $gatewayPC = 'https://www.99bill.com/gateway/recvMerchantInfoAction.htm';
       private $gatewayMobile = 'https://www.99bill.com/mobilegateway/recvMerchantInfoAction.htm';
       private $pcarduserPEMFile = 'certs/pcarduser.pem.php';
       
       private $payParamStr = '';
       private $signMsg = '';
       
       private $payParamsOfPC = [
           'inputCharset' => '1',
           'pageUrl' => '',
           'bgUrl' => '',
           'version' => 'v2.0',
           'language' => '1',
           'signType' => '4',
           'signMsg' => '',
           'merchantAcctId' => '',
           'payerName' => '',
           'payerContactType' => '',
           'payerContact' => '',
           'orderId' => '',
           'orderAmount' => '',
           'orderTime' => '',
           'productName' => '',
           'productNum' => '',
           'productId' => '',
           'productDesc' => '',
           'ext1' => '',
           'ext2' => '',
           'payType' => '',
           'bankId' => '',
           'redoFlag' => '',
           'pid' => '',
       ];
       
       private $payParamsOfMobile = [
           'inputCharset' => '1',
           'pageUrl' => '',
           'bgUrl' => '',
           'version' => 'mobile1.0',
           'language' => '1',
           'signType' => '4',
           'merchantAcctId' => '',
           'payerName' => '',
           'payerContactType' => '1',
           'payerContact' => '',
           'payerIdType' => '3',    // 指定付款人
           'payerId' => '',
           'orderId' => '',
           'orderAmount' => '',
           'orderTime' => '',
           'productName' => '',
           'productNum' => '',
           'productId' => '',
           'productDesc' => '',
           'ext1' => '',
           'ext2' => '',
           'payType' => '21',    // 快捷支付
           'bankId' => '',
           'redoFlag' => '0',
           'pid' => '',
       ];
       
       public function __construct($config)
       {
           date_default_timezone_set('Asia/Shanghai');
           $this->payParamsOfPC['bgUrl']   = $this->payParamsOfMobile['bgUrl']   = $config['return_url'];
           $this->payParamsOfPC['pageUrl'] = $this->payParamsOfMobile['pageUrl'] = $config['notify_url'];
           $this->payParamsOfPC['productNum'] = $this->payParamsOfMobile['productNum'] = $config['goodsNum'] ? $config['goodsNum'] : 1;
           $this->payParamsOfPC['orderTime']  = $this->payParamsOfMobile['orderTime'] = date('YmdHis');
       }
       
       // by rmb_demo_php
       public function kq_ck_null($kq_va, $kq_na)
       {
           if ($kq_va == "") {
               return $kq_va="";
           } else {
               return $kq_va=$kq_na.'='.$kq_va.'&';
           }
       }
       
       public function buildPayParams($isMobile)
       {
           $payParamsArr = $isMobile ? $this->payParamsOfMobile : $this->payParamsOfPC;
           $payParamStr = '';
           foreach ($payParamsArr as $k => $v) {
               $payParamStr .= $this->kq_ck_null($v, $k);
           }
           $payParamStr = $isMobile
               ? rtrim($payParamStr, '&')
               : mb_substr($payParamStr, 0, mb_strlen($payParamStr)-1);
           $this->payParamStr = $payParamStr;
           return $this->payParamStr;
       }
       
       public function signPayParams()
       {
           $priv_key = trim(require_once __CORE_DIR.$this->pcarduserPEMFile);
   //        $fp = fopen(__CORE_DIR.$this->pcarduserPEMFile, 'r');
   //        $priv_key = fread($fp, 123456);
   //        fclose($fp);
           $pkeyid = openssl_get_privatekey($priv_key);
           openssl_sign($this->payParamStr, $signMsg, $pkeyid, OPENSSL_ALGO_SHA1);
           openssl_free_key($pkeyid);    // free the key from memory
           $this->signMsg = base64_encode($signMsg);
           return $this->signMsg;
       }
       
       public function build_url($params)
       {
           if (!$params) {
               return false;
           }
           $isMobile = (!isset($params['gatewayType']) || (isset($params['gatewayType'])&&$params['gatewayType'] != 'pc'))
               ? true
               : false;
           $detail = K::M('payment/payment')->payment($this->code);
           if ($isMobile) {
               $gateway = $detail['kq_gateway_mobile']
                   ? $detail['kq_gateway_mobile']
                   : $this->gatewayMobile;
               $payParamsType = 'payParamsOfMobile';
           } else {
               $gateway = $detail['kq_gateway_pc']
                   ? $detail['kq_gateway_pc']
                   : $this->gatewayPC;
               $payParamsType = 'payParamsOfPC';
           }
           $payParamsTypeArr = &$this->$payParamsType;
           $payParamsTypeArr['merchantAcctId'] = $detail['config']['kq_merchant_id'];
           $payParamsTypeArr['productName'] = $params['title'];
           $payParamsTypeArr['productDesc'] = $params['body'];
           $payParamsTypeArr['payerName']   = $params['payerName'];
           $payParamsTypeArr['payerId']     = $params['uid'];
           $payParamsTypeArr['orderAmount'] = $params['amount'];
           $payParamsTypeArr['orderId']     = $params['trade_no'];
           $this->buildPayParams($isMobile);
           $this->signPayParams();
           $payParamsTypeArr['signMsg'] = $this->signMsg;
           $payParamsTypeArr['gateway'] = $gateway;
           $payParamsTypeArr['entry_url'] = '/trade/kuaiqian';
           return $payParamsTypeArr;
       }
       
       public function build_app($params)
       {
           return $this->build_url($params);
       }
       
       public function build_form($params)
       {
           return $this->build_url($params);
       }
   } 
```

其中，`build_app` 和 `build_url` 是为了和其他支付接口保持一致，这样才可以通过现有的代码来在控制器调用快钱支付。



# 支付

由于快钱支付接口会检查 `HTTP Referrer` 字段，虽然可以通过在 CURL 中设置，但是我考虑到本来就要在浏览器页面发起支付请求，因此我用了最原始的表单方式来发起支付请求。

```
    <!-- themes/default/kuaiqian/send.html -->
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>正在使用快钱支付</title>
    </head>
    <body style="display:none">
    
    <form method='GET' action="<{$gateway}>">
        <input type="hidden" name="inputCharset" value="<{$inputCharset}>">
        <input type="hidden" name="pageUrl" value="<{$pageUrl}>">
        <input type="hidden" name="bgUrl" value="<{$bgUrl}>">
        <input type="hidden" name="version" value="<{$version}>">
        <input type="hidden" name="language" value="<{$language}>">
        <input type="hidden" name="signType" value="<{$signType}>">
        <input type="hidden" name="merchantAcctId" value="<{$merchantAcctId}>">
        <input type="hidden" name="payerName" value="<{$payerName}>">
        <input type="hidden" name="payerContactType" value="<{$payerContactType}>">
        <input type="hidden" name="payerContact" value="<{$payerContact}>">
        <input type="hidden" name="payerIdType" value="<{$payerIdType}>">
        <input type="hidden" name="payerId" value="<{$payerId}>">
        <input type="hidden" name="orderId" value="<{$orderId}>">
        <input type="hidden" name="orderAmount" value="<{$orderAmount}>">
        <input type="hidden" name="orderTime" value="<{$orderTime}>">
        <input type="hidden" name="productName" value="<{$productName}>">
        <input type="hidden" name="productNum" value="<{$productNum}>">
        <input type="hidden" name="productId" value="<{$productId}>">
        <input type="hidden" name="productDesc" value="<{$productDesc}>">
        <input type="hidden" name="ext1" value="<{$ext1}>">
        <input type="hidden" name="ext2" value="<{$ext2}>">
        <input type="hidden" name="payType" value="<{$payType}>">
        <input type="hidden" name="bankId" value="<{$bankId}>">
        <input type="hidden" name="redoFlag" value="<{$redoFlag}>">
        <input type="hidden" name="pid" value="<{$pid}>">
        <input type="hidden" name="signMsg" value="<{$signMsg}>">
        <input type="submit" value="SUBMIT" id="payNow">
    </form>
    </body>
    </html>
    <script>
        document.getElementById('payNow').click();
    </script>
```

这样无论是什么业务需要使用快钱支付，只需要通过这个支付页面就行了，不需要像使用 CURL 时去判断到底是用哪个 URL 地址。


# 回调

支付完成后，快钱会发起两种回调：

- 一个 `REQUEST` 到后台处理支付结果的回调URL，用于验签、根据支付结果逻辑处理；

## 注意事项

这个回调处理完逻辑后，必须返回 `result` 和 `redirecturl` XML 标签对，否则快钱会一直请求。

此外，返回的 `<result></result>` 标签值必须是验签的结果：`0` 或 `1`。

- 一个 `GET` 支付结果页面，显示支付结果。

这个显示页面就是由上面 `redirecturl` 指定的，可含自定义参数用于定制提示信息。

## 验签

除了这两个文件外，还有一些工具性方法，被我封装到了 `system/models/tools/tool.mdl.php`。相关方法是：

```
    // [快钱支付回调参数校验]
    // re-packing from kuaiqian demo for mobile
    public function kqPayCallbackSignVerify($params, $cert)
    {
      // by rmb_demo_php @kuaiqian
      function kq_ck_null($kq_va, $kq_na) {
        if ($kq_va == "") {
          return $kq_va="";
        } else {
          return $kq_va=$kq_na.'='.$kq_va.'&';
        }
      }
      
      //人民币网关账号，该账号为11位人民币网关商户编号+01,该值与提交时相同。
      $kq_check_all_para=kq_ck_null($params['merchantAcctId'], 'merchantAcctId');
      
      //网关版本，固定值：v2.0,该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['version'], 'version');
      
      //语言种类，1代表中文显示，2代表英文显示。默认为1,该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['language'], 'language');
      
      //签名类型,该值为4，代表PKI加密方式,该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['signType'], 'signType');
      
      //支付方式，一般为00，代表所有的支付方式。如果是银行直连商户，该值为10,该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['payType'], 'payType');
      
      //银行代码，如果payType为00，该值为空；如果payType为10,该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['bankId'], 'bankId');
      
      //商户订单号，,该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['orderId'], 'orderId');
      
      //订单提交时间，格式：yyyyMMddHHmmss，如：20071117020101,该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['orderTime'], 'orderTime');
      
      //订单金额，金额以“分”为单位，商户测试以1分测试即可，切勿以大金额测试,该值与支付时相同。
      $kq_check_all_para.=kq_ck_null($params['orderAmount'], 'orderAmount');
      $kq_check_all_para.=kq_ck_null($params['bindCard'], 'bindCard');
      $kq_check_all_para.=kq_ck_null($params['bindMobile'], 'bindMobile');
      
      // 快钱交易号，商户每一笔交易都会在快钱生成一个交易号。
      $kq_check_all_para.=kq_ck_null($params['dealId'], 'dealId');
      
      //银行交易号 ，快钱交易在银行支付时对应的交易号，如果不是通过银行卡支付，则为空
      $kq_check_all_para.=kq_ck_null($params['bankDealId'], 'bankDealId');
      
      //快钱交易时间，快钱对交易进行处理的时间,格式：yyyyMMddHHmmss，如：20071117020101
      $kq_check_all_para.=kq_ck_null($params['dealTime'], 'dealTime');
      
      //商户实际支付金额 以分为单位。比方10元，提交时金额应为1000。该金额代表商户快钱账户最终收到的金额。
      $kq_check_all_para.=kq_ck_null($params['payAmount'], 'payAmount');
      
      //费用，快钱收取商户的手续费，单位为分。
      $kq_check_all_para.=kq_ck_null($params['fee'], 'fee');
      
      //扩展字段1，该值与提交时相同
      $kq_check_all_para.=kq_ck_null($params['ext1'], 'ext1');
      
      //扩展字段2，该值与提交时相同。
      $kq_check_all_para.=kq_ck_null($params['ext2'], 'ext2');
      
      //处理结果， 10支付成功，11 支付失败，00订单申请成功，01 订单申请失败
      $kq_check_all_para.=kq_ck_null($params['payResult'], 'payResult');
      
      //错误代码 ，请参照《人民币网关接口文档》最后部分的详细解释。
      $kq_check_all_para.=kq_ck_null($params['errCode'], 'errCode');
      $trans_body = mb_substr($kq_check_all_para, 0, mb_strlen($kq_check_all_para)-1);
      $MAC = base64_decode($params['signMsg']);
      //        $fp = fopen("./99bill[1].cert.rsa.20140803.cer", "r");
      //        $cert = fread($fp, 8192);
      //        fclose($fp);
      $pubkeyid = openssl_get_publickey($cert);
      return openssl_verify($trans_body, $MAC, $pubkeyid);
    }
```

# 附录

- **快钱返回值**

```
    {
    	"dealTime": "20170623160809",
    	"payAmount": "100",
    	"bindMobile": "1593640",
    	"signType": "4",
    	"bindCard": "6214832276",
    	"errCode": "",
    	"merchantAcctId": "1020092607701",
    	"orderTime": "20170623160616",
    	"dealId": "2564166292",
    	"version": "mobile1.0",
    	"bankId": "CMB",
    	"fee": "",
    	"bankDealId": "170623732597",
    	"ext1": "",
    	"payResult": "10",
    	"ext2": "",
    	"orderAmount": "100",
    	"signMsg": "U63zh45aT3qMBr9dA1QHTr9biG3+VvpVJgD0ZsPPKatLF6uFPkWgIA66ffqMRZYG+3w3jwajlwBhvq+2dHn0PL9+BM9ab6bhcLmRsuEvC93fsq3rQzcreEGQyyWKb5HFUQVqnVE7xgFEG\/5ie\/xywdtL0hQJl2yGzxjEjUXUtKSTH7+7nN2VaqLe69Z5yZP0il5G7FWcuahekfxDGUmG8n8yiqH9+f8TbJYLT0jNB9AXgrwfLwmidyrkp7Ar6pWpVXJ65CbwUrZ+JQP5Li1pMpHjhj\/bCP2Xi8L8UzLZAwxbUxV+UN3RB5BzuK4kbE0\/4tqlh9HoZAVu4rwP6aXEPA==",
    	"payType": "21-1",
    	"language": "1",
    	"orderId": "1706233980"
    }
```

# FAQ

- 为什么快钱请求回调URL时原样输出两个标签了？

被登录验证挡住了。

# 其他

- `json_decode()` 会把 `+` 解码成空格

# 参考

- <u>[快钱开放平台-接入中心](http://open.99bill.com/menu!access.do#)</u>
