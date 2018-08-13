---
title: 云通信 「开发日志」
date: 2018-02-02 13:28:55
categories: ThirdParty
tags: [阿里云, Aliyun, 云通信, ThirdParty]
---

> 阿里云通信, 原阿里大于. 大神直接绕路,本文乃小学生式教学.

本文只是使用了一个短信通知服务,如需要别的服务可:[查看文档](https://help.aliyun.com/document_detail/59210.html?spm=a2c4g.11186623.6.544.y1MZHh)


<!--more-->

**简单来说就下下图:**

![短信使用流程](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/59210/cn_zh/1512460650087/88.png)

# 准备

- 阿里云账号, 毫无疑问.

- 开通 <u>[短信服务](https://dysms.console.aliyun.com/dysms.htm?spm=5176.sms-account.101.125.36397954VtIeVu#/overview)</u>

- <u>[下载SDk](https://help.aliyun.com/document_detail/55451.html?spm=5176.10629532.106.2.2e4fa65b55fFpB)</u>

我使用的是PHP,当然下载PHP SDK

- 申请阿里云的访问密钥 <u>[Access Key](https://ak-console.aliyun.com/?spm=a2c4g.11186623.2.5.4jPy2R#/accesskey)</u>

- 获取参数

  - <u>[短信签名](https://dysms.console.aliyun.com/dysms.htm?spm=5176.8195934.907839.sms8.1a39d8b4IkjUKx#/develop/sign)
  - <u>[短信模板](https://dysms.console.aliyun.com/dysms.htm?spm=5176.8195934.907839.sms8.1a39d8b4IkjUKx#/develop/template)</u>
  
**短信签名**,个人只能拥有一个,需要多个需要升级为企业.**短信模板**可以申请多个.

- 入参列表/出参列表 [详见文档](https://help.aliyun.com/document_detail/55451.html?spm=5176.10629532.106.2.2e4fa65b55fFpB)


# 使用

```
    // 代码节选，详见api_demo/SmsDemo.php
    ...
    class SmsDemo
    {
        ...
     * 短信服务API产品的DEMO程序,工程中包含了一个SmsDemo类，直接通过
     * 执行此文件即可体验语音服务产品API功能(只需要将AK替换成开通了云通信-短信服务产品功能的AK即可)
     * 备注:Demo工程编码采用UTF-8
     */
    /**
         * 发送短信
         * @return stdClass
         */
        public static function sendSms() {
        
            // 初始化SendSmsRequest实例用于设置发送短信的参数
            $request = new SendSmsRequest();
            
            // 必填，设置短信接收号码
            $request->setPhoneNumbers("12345678901");
            
            // 必填，设置签名名称，应严格按"签名名称"填写，请参考: https://dysms.console.aliyun.com/dysms.htm#/develop/sign
            $request->setSignName("短信签名");
            
            // 必填，设置模板CODE，应严格按"模板CODE"填写, 请参考: https://dysms.console.aliyun.com/dysms.htm#/develop/template
            $request->setTemplateCode("SMS_0000001");
            
            // 可选，设置模板参数, 假如模板中存在变量需要替换则为必填项
            $request->setTemplateParam(json_encode(Array(  // 短信模板中字段的值
                "code"=>"12345",
                "product"=>"dsd"
            )));
            
            // 可选，设置流水号
            $request->setOutId("yourOutId");
            
            // 选填，上行短信扩展码（扩展码字段控制在7位或以下，无特殊需求用户请忽略此字段）
            $request->setSmsUpExtendCode("1234567");
            
            // 发起访问请求
            $acsResponse = static::getAcsClient()->getAcsResponse($request);
            
            return $acsResponse;
        }
        
    // 调用示例：
    set_time_limit(0);
    header('Content-Type: text/plain; charset=utf-8');
    
    $response = SmsDemo::sendSms();
    echo "发送短信(sendSms)接口返回的结果:\n";
    print_r($response);
```

> 阿里云的服务,文档都很人性化,花点时间看看即可上手

# 进阶

将代码上传到服务器,使用Crontab 定时任务,每天定时发短信~~


# FAQ

**记得充钱,云通信短信服务是收费的,不过巨便宜~~**


# 参考

- [短信发送API(SendSms)](https://help.aliyun.com/document_detail/55451.html?spm=5176.10629532.106.2.2e4fa65b55fFpB)
- [签名申请手册](https://help.aliyun.com/document_detail/55327.html?spm=a2c4g.11186623.2.6.YAjI2N)
- [模板申请手册](https://help.aliyun.com/document_detail/55330.html?spm=a2c4g.11186623.2.7.6qpTsI)
- [Crontab Is So Easy](http://blog.caoxl.com/2018/01/24/Crontab-Is-So-Easy/)