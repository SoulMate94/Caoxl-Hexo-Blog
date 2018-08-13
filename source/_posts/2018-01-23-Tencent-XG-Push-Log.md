---
title: 腾讯信鸽推送 「开发日志」
date: 2018-01-23 14:54:33
categories: ThirdParty
tags: [信鸽, 推送, 第三方, 腾讯, ThirdParty]
---

> 简单记录腾讯信鸽推送开发日志
> > <u>[腾讯信鸽推送 文档](http://docs.developer.qq.com/xg/)</u>

# iOS 推送工作原理

## APNs

`Apple Push Notification Service`，直译过来就是“苹果推送通知服务”，即苹果专门的推送服务器。

由于 Apple 对设备的控制非常严格，iOS 内应用消息的推送必须要经过 APNs。

<!--more-->

## 被推送消息的转发顺序

***应用服务器（Provider）=> APNs => iPhone／iOS => App。***

![转发顺序](https://segmentfault.com/image?src=https://blog.avoscloud.com/wp-content/uploads/2014/05/registration_sequence_2x.png&objectId=1190000000520755&token=5bf9906773e763d2e743a8f503f71556)


## 工作原理说明

> 苹果利用自己专门的`推送服务器（APNs）`接收来自我们自己的应用服务器的需要被推送的信息，然后推送到`指定的iOS设备`上，然后由设备通知到我们的应用程序，设备以通知或者声音的形式通知用户有新的消息。
> 
> **推送的前提是装有我们应用的设备需要向APNs服务器注册**，注册成功后APNs服务器会返给我们一个device_token，拿到这个token后我们将这个token发给我们自己的应用服务器，当有需要被推送的消息时，我们的应用服务器会将消息按指定的格式打包，然后结合设备的device_token一并发给APNs服务器，由于我们的应用和APNs维持一个基于TCP的长连接，APNs将新消息推送到我们设备上，然后在屏幕上显示出新消息来。


## 推送数据格式

`JSON`，且 APNs 限制了每个 `notification` 的 `payload` 最大长度是 **256** 字节。举例：

```
    {
        "aps": {
            "alert": "老板赞了你", 
            "sound": "default",
    	    "badge": 1;
        }, 
        "msg": "这里是额外消息，不显示在通知界面上。"
      	"action": {
      		"type": 2
    	}
    }
```

- `alert` 的内容就是会显示在用户手机上的推送信息，它的值可以是一个 `String`，也可以是一个 `JSON`。当它是文本消息的时候，系统就会把这些文字显示到一个 `alertView `中；如果它也是由一个 `JSON` 组成的话，其包含的字段如下：
  
  - body
  - action-loc-key
  - loc-key
  - loc-args
  - launch-image
  
  body 部分就是 `alertView` 中将要展现出来的文本消息，loc-* 属性主要是用来实现**本地化**消息，`launch-image` 只是app 主 bundle 里的一个图片文件的名称，一般来说我们不指定这一属性。
  - `badge` 是会在应用 icon 右上角显示的数量（注意是整型），提示有多少条未读消息等。
  - `sound` 就是当推送信息送达是手机播放的声音，传 `defalut` 就标明使用系统默认声音，如果传比如 `beep.wav` 就会播放在我们应用工程目录下名称为 *beep.wav* 的音频文件。
  - `aps` 之外的可以自定义数据结构。
  
## 本地化

> 有两种办法可以实现推送消息的本地化：
  1，在推送的payload中使用`loc-key`和`loc-args`来指定进行本地化，这样Provider方只需要按照统一的格式来发送即可，消息的解析和组装则由客户端来完成。
  2，如果推送的payload里面不包含`loc-key/loc-args`信息，那么Provider方就需要自己做本地化处理，然后给不同的device发送不同的消息，为了做到这一点，还需要app在上传device token的时候也把用户的语言设置信息传回来。
  
# 腾讯信鸽接入

## 客户端配置

这个主要是 iOS 去完成的，这里略过，详见：<u>[iOS接入-腾讯信鸽](http://developer.qq.com/wiki/xg/iOS%E6%8E%A5%E5%85%A5/iOS%20%E8%AF%81%E4%B9%A6%E8%AE%BE%E7%BD%AE%E6%8C%87%E5%8D%97/iOS%20%E8%AF%81%E4%B9%A6%E8%AE%BE%E7%BD%AE%E6%8C%87%E5%8D%97.html)</u>。


## 服务端配置

服务端接入及调用腾讯信鸽非常简单，可以通过提供的服务端 SDK，也可以直接使用提供的 `RESTful API`。下面以在 `Lumen` 中集成腾讯信鸽的 `PHP SDK` 为例说明。

- 引入类文件

腾讯信鸽的 `PHP SDK` 主要只有一个类文件 `XingeApp.php`。把它放在 `app/Tools/XingeApp.php` 下，命名空间为设置为：`namespace App\Tools`。

- 配置 `access_id` 和 `secret_key`

相应的配置文件放在 `config/txxg.php` 中，内容只有以下两个：

```
    <?php
    /*
    |--------------------------------------------------------------------------
    | 腾讯信鸽配置
    |--------------------------------------------------------------------------
    |
    | Add this config file in .gitignore and put it on the prod server manually
    |
    */
    return [
        'access_id' => '1024',
        'secret_key' => 'xxxx',    // 32 bit(md5)
    ];
```

**`access_id` 和 `secret_key` 在注册了腾讯信鸽后会提供，填在这里就行了。**

- 在工具类中调用 SDK 中的方法

```
    namespace App\Tools;
    
    class Tol
    {
      public static function iosPush($account, $msg, $msg_num)
        {
            $txxg_path = base_path().'/config/txxg.php';
            $txxg      = false;
            
            if (file_exists($txxg_path))
                $txxg = include_once $txxg_path;
                
            if (!$txxg ||
                !$txxg['access_id'] ||
                !$txxg['secret_key']) abort(
                500,
                '腾讯信鸽配置不存在或不正确(config/txxg.php)'
            );
            
            list(
                $accessId,
                $secretKey,
                ) = array_values($txxg);
                
            return XingeApp::PushAccountIos(
                $accessId,
                $secretKey,
                $msg,
                $account,
                XingeApp::IOSENV_PROD,
                $msg_num
            );
        }
    }
```

之后在控制器中如果要给 `App` 发送推送消息，则只需要执行 `Tol::iosPush()` 就可以了。（注意引入该工具类先）

# 参考

- [信鸽开发者手册](http://developer.qq.com/wiki/xg/%E4%BF%A1%E9%B8%BD%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D/%E4%BF%A1%E9%B8%BD%E5%9F%BA%E7%A1%80%E4%BB%8B%E7%BB%8D.html)
- [下载 SDK-腾讯信鸽](http://xg.qq.com/xg/ctr_index/download.md)
- [苹果推送(APNs)ios push小结](https://www.lvtao.net/ios/apple-ios-push.html)
- [细说 iOS 消息推送](https://segmentfault.com/a/1190000000520755)
- [移动手机消息推送机制](http://blog.csdn.net/zphappy/article/details/6658504)
- [一步一步实现iOS应用PUSH功能](http://tanqisen.github.io/blog/2013/02/27/ios-push-apns/)
- [极光官方文档](http://docs.jiguang.cn/)