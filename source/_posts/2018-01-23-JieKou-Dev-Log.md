---
title: 接口网 短信接口「开发日志」
date: 2018-01-23 14:32:06
categories: ThirdParty
tags: [接口网, 短信接口, 第三方, ThirdParty]
---

> <u>[接口网短信接口](http://www.106jiekou.com/)</u>


# 接口地址

不同的编码方式请使用不同的接口地址，以 UTF-8 举例说明：

- 国内「非群发」类短信接口格式

```
    http://sms.106jiekou.com/utf8/sms.aspx?account=用户账号&password=接口密码&mobile=号码&content=您的订单编码：888888。如需帮助请联系客服。
```

- 国内「群发」类短信接口格式（不支持单条频繁提交）

```
    http://sms.106jiekou.com/utf8/openapi.aspx?account=用户账号&password=接口密码&mobile=号码&content=内容
```

- 国际「非群发」类短信接口格式

```
    http://sms.106jiekou.com/utf8/worldapi.aspx?account=用户账号&password=接口密码&mobile=号码&content=内容
```

<!--more-->

短信接口都是 `JSON` 响应。

此外，`GBK` 接口地址和 `UTF-8` 的区别只是把上面的 `utf-8` 改为 `gbk` 即可。

- 查询余额（XML 响应）

```
    http://www.dxton.com/webservice/sms.asmx/GetNum?account=用户账号&password=接口密码
```

# 接口参数

## 接口账户和接口密码

接口账户就是登录接口网的账户，**但接口密码不是登录密码**，需要自行在后台设置。

## 接收号码

- 频率限制

每个号码接收频率 5 次/天。

> 不然易引起运营商网关屏蔽数天，如果需要更高频率联系客服申请。

- 群发

当调用群发接口时，使用英文逗号 `,` 分隔不同的手机号码即可，比如：`13333333333`,`15555555555`。**国际建议 100 个内，国内建议 1000 个内**。

- 国际

需要注意带上国际区码和地区区码。比如

```
    国际区码：49
    地区区码：0179
    手机号码：233333
```

则正确的格式为：`49179233333`。**注意，地区城市区码的 0或 00 不用加。**

## 短信内容

发送内容需要进行 `URL` 字符标准化转码。

有一个专用调试模版：您的订单编码：`888888`。如需帮助请联系客服。。

> URL 字符编码说明：返回字符串，此字符串中除了 `-/_/.` 之外的所有非字母数字字符，都将被替换成百分号 `%` 后跟两位十六进制数，空格则编码为加号 `+`。

# 短信模版

短信模版指的是发送到用户手机时看到的内容所采用的格式。

模版中含有「变量」，变量是在项目根据需要动态生成的不同内容。变量套用被允许使用的模版，生成最终发送给用户的短信内容，连同用户手机号码，通过参数的形式调用接口网接口，发送给目标用户。

**短信模版举例如下：**

- 未充值用户调试专用短信模板

```
    尊敬的用户您已经注册成功，用户名：`<账号用户名>` 密码：`<账号密码>` 感谢您的注册！
```

- 专用调试模版

```
    您的订单编码：888888。如需帮助请联系客服。
```

- 普通模版

```
    您的订单编码：【变量】。如需帮助请联系客服。
```

其中， **「变量」可以自定义内容，其它都要和模板一致，包括空格、标点符号等。**

## 限制

- 小批量调试或应用订购，短信内容只能用“系统公共模版”相对固定格式，否则会报错104（内容未审核）
- 自定义短信模版、签名，需要审批和备案

> 由于短信行业受运营商政策管制，只有政策允许且你的帐号级别够的时候，才支持。若帐号无权限或系统未开放时，只能选用系统默认的通用性短信签名和短信模版。如果都支持，编辑好签名、模板，后台提交，我们报备运营商，审批下来后就可以按贵公司的格式进行发送了。（以后台通告为准）
>
> 提交时，签名不用加 【】 号，模板中属于变量的地方用 【变量】代替，否则系统识别不出来。

## 短信签名

短信末尾或前面跟的品牌签名。比如：

```
    您的订单编码：888888。如需帮助请联系客服。【会员验证通知】
```

其中，`”会员验证通知”` 就属于`短信签名`，一般是公司或产品简称。

## 小批量应用【部分自定义内容和签名】实现方法

- 可以把订单编码、客服热线、【店铺名称】，都融入到 一个【变量】里。

**短信模板：** 您的订单编码：【变量】。如需帮助请联系客服。

**实际内容：** 您的订单编码：20140208 客服热线4006668280【速度网络】。如需帮助请联系客服。【会员验证通知】

- 可以把验证码、网站或软件名称，都融入到 一个【变量】里。

**短信模板：** 您的订单编码：【变量】。如需帮助请联系客服。

**实际内容：** 您的订单编码：8888【速度网络】。如需帮助请联系客服。【会员验证通知】

# 返回状态码及其说明

```
    100			发送成功
    101			验证失败
    102			手机号码格式不正确
    103			会员级别不够
    104			内容未审核
    105			内容过多
    106			账户余额不足
    107			Ip受限
    108			手机号码发送太频繁，请换号或隔天再发
    109			帐号被锁定
    110			发送通道不正确
    111			当前时间段禁止短信发送
    120			系统升级
```


# 相关代码

```
    <?php
    
    namespace App\Tools;
    
    class DxtonSMS
    {
        protected $send_api = 'http://sms.106jiekou.com/utf8/sms.aspx';
        protected $sms_tpls = [
            'default' => '您的订单编码：CODE。如需帮助请联系客服。'
        ];
        
        protected $account = null;
        protected $api_key = null;
        protected $mobiles = null;
        protected $code    = null;
        protected $tpl     = null;
        protected $api_url = null;
        protected $content = null;
        
        public function __construct(
            $mobiles,
            $code,
            $tpl = 'default',
            $account = null,
            $api_key = null
        ) {
            list(
                $this->account,
                $this->api_key,
                $this->mobiles,
                $this->code,
                $this->tpl
            ) = [
                $account,
                $api_key,
                $mobiles,
                $code,
                $tpl,
            ];
            
            if (is_null($this->account) || is_null($this->api_key)) {
                self::setDefaultAuth();
            }
            
            self::setApiUrl();
        }
        
        public static function getContent()
        {
            //
        }
        
        public static function setApiUrl(
            $account,
            $password,
            $mobile,
            $code,
            $tpl
        ) {
            // account=帐号&password=接口密码&mobile=手机号码&content=".rawurlencode(self::getContent());
        }
        
        public static function setDefaultAuth()
        {
            try {
                $this->account = env('DXT_ACCOUNT');
                $this->api_key = env('DXT_API_KEY');
            } catch (\Exception $e) {
                die($e->getMessage());
            }
        }
        
        public static function getSendResBySock($data, $target)
        {
            $url_info    = parse_url($target);
            $httpheader  = "POST " . $url_info['path'] . " HTTP/1.0\r\n";
            $httpheader .= "Host:" . $url_info['host'] . "\r\n";
            $httpheader .= "Content-Type:application/x-www-form-urlencoded\r\n";
            $httpheader .= "Content-Length:" . strlen($data) . "\r\n";
            $httpheader .= "Connection:close\r\n\r\n";
            // $httpheader .= "Connection:Keep-Alive\r\n\r\n";
            $httpheader .= $data;
            
            $fd = fsockopen($url_info['host'], 80);
            fwrite($fd, $httpheader);
            $gets = "";
            
            while (!feof($fd)) {
                $gets .= fread($fd, 128);
            }
            
            fclose($fd);
            return $gets;
        }
        
        public static function getSendResByCurl($api_url, $content)
        {
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_URL, $api_url);
            curl_setopt($ch, CURLOPT_HEADER, false);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            curl_setopt($ch, CURLOPT_NOBODY, true);
            curl_setopt($ch, CURLOPT_POST, true);
            curl_setopt($ch, CURLOPT_POSTFIELDS, $content);
            
            $send_res = curl_exec($ch);
            
            if (curl_errno($ch) !== 0) {
                $send_res = false;
            }
            
            curl_close($ch);
            
            return $send_res;
        }
    }
```

# FAQ

- 短信长度如何收费？

70 字符按 1 条收费，超 70 字，按 65 /条，多条收费。(通道行业标准，部分通道 67 字符单条）

- 可以绑定IP吗？

为了避免各种原因的帐号盗用情况，造成贵公司短信被滥用，所以我们的系统有ip验证功能，只发送您这边认可的ip地址提交的短信。

- 接口调试返回“104 内容未审核”错误？

发送内容未按系统模板或备案的模板下发；或程序编码格式和接口不一致。



# 参考

- [接口网短信接口开发记录](http://blog.cjli.info/2016/07/07/106jiekou-Dev-Log/)