---
title: 微信公众号「开发日志」
date: 2018-01-23 8:59:13
categories: ThirdParty
tags: [WeChat, 微信, 第三方, 公众号, ThirdParty]
---

> 简单总结下微信 Web 开发流程。
> > 该博客配合微信公众号开发文档使用。

<!--more-->

# 基本概念

## 前提

- 熟悉服务器后端开发
- 有公网 IP 且 `80 端口`（http）或 `443 端口（https）`可访问的服务器


## 简介

- 基本认识

  - 订阅号
  主要偏于为用户传达资讯（类似报纸杂志），认证前后都是每天只可以群发一条消息。
  
  - 服务号
  主要偏于服务交互（类似银行，114，提供服务查询），认证前后都是每个月可群发4条消息。
  
  - 企业号
  主要用于公司内部通讯使用，需要先有成员的通讯信息验证才可以关注成功企业号。
  
> 1、如果想简单的发送消息，达到宣传效果，建议可选择**订阅号**；
  2、如果想进行商品销售，进行商品售卖，建议可申请**服务号**；
  3、如果想用来管理内部企业员工、团队，对内使用，可申请**企业号**。
  4、订阅号可通过微信认证资质审核通过后有一次升级为服务号的入口，升级成功后类型不可再变。
  5、*服务号不可变更成订阅号*。
  <u>http://kf.qq.com/faq/120911VrYVrA130805byM32u.html</u>

- 申请流程

> 到 <u>https://mp.weixin.qq.com</u> 点击「立即注册」后，按需求和流程注册即可。

- 工作原理

整个过程只有 3 个角色：`用户`、`微信`和`公众号`。

用户通过微信端入口，把各种请求发送到微信服务器，微信服务器将用户请求封装成一定格式的数据包，以 `GET/POST` 的方式转发到发到公众号所在的应用服务器，应用服务器解析数据包，以完成相应的逻辑，并将处理结果返回按照一定的格式返回给微信，微信再返回给用户。

- API 介绍

> <u>[微信公众平台API文档](https://mp.weixin.qq.com/wiki.)</u>

- 微信界面交互接口/微信公众号的使用模式

有两种定义方式：

1、通过**界面/编辑模式：** 登录微信公众号后，选择 「功能」=>「自定义菜单」
2、通过**开发者接口/开发者模式：**  登录微信公众号后，选择 「开发」=>「基本配置」=> 「启用」

两种只能选择其一。**要进入微信平台的编辑模式需要先关闭开发者模式**，但是编辑模式的功能远不如开发者模式。

- 事件推送
  - 推送类型介绍
    - 关注和取消关注
    - 订阅事件推送
  - 如何响应事件推送
- 数据交互

XML。

- 模版消息：详见手册。
- 消息收发：详见手册。
- 网页授权：详见手册。


## 接入应用

### 填写服务器配置

- `URL`

是正在开发的应用，所运行的业务服务器上，用来接收微信消息和事件的接口地址。

必须是一个具体的接口地址，而不是只填写服务器 IP，比如：`http://www.myapp.com/weixin.php。`

- `Token`

用于接入微信时，生成签名以验证安全性使用。内容可自定义。

该 Token 一般保存在应用服务器上，当微信服务器请求应用服务器接口 URL 时，连同请求中包含的 `timestamp`、`nonce` 一起生成一个签名，并和请求中的 `signature` 对比。

- EncodingAESKey

是一个秘钥，用于加解密微信服务器和应用服务器之间传递的消息体。可自定义，也可由微信随机生成。

- 消息加解密方式

有三种选择，详见微信公众号设置，一般开发期选择「兼容模式」。

模式的选择，与服务器配置在提交后都会立即生效。

### 验证消息是否来自微信服务器

这个是验证，对上一步服务器配置中的 URL 请求来源，是否是微信服务器。

服务器配置好提交后，微信服务器会发送一个 `GET` 请求到这个 `URL`。

这个请求中包含了 4 个参数，使用这 4 个参数校验的过程示例如下：

```
    function weixinSignatureCheck() {
      $signature = $_GET['signature'];
      $timestamp = $_GET['timestamp'];
      $nonce     = $_GET['nonce'];
      $echostr   = $_GET['echostr'];
      
      $token     = TOKEN;    // 假设这个常量已经正确代表了上述配置中填写的 Token
      $tmp_arr   = [$timestamp, $nonce, $token];
      sort($tmp_arr) ;
      // 加密／校验流程
      // 1. 将token、timestamp、nonce三个参数进行字典序排序
      // 2. 将三个参数字符串拼接成一个字符串进行sha1加密
      // 3. 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信
      $tmp_str   = sha1(implode($tmp_arr));
      
      // 若确认此次GET请求来自微信服务器，请原样返回echostr参数内容，则接入生效，成为开发者成功，否则接入失败
      if ($signature == $tmp_str) {
        echo $echostr;
      }
    }
```

### 根据接口文档实现相应需求

当开发者身份验证通过之后，即代表我们自己的应用服务器接入微信成功，下次请求就不会再传入 `echostr` 这个参数了。

之后，便可以使用开发者身份调用微信公众号提供的接口完成相应业务逻辑了。

## 通信格式

微信服务器转发过来的消息包格式统一为 XML，我们的服务器需要自行解析数据包，根据数据包的不同内容完成相应的逻辑。

```
    / 微信入口
    function index() {
      if (isset($_GET['echostr'])) {
        weixinSignatureCheck();
      } else {
        response();
      }
    }
    
    // 获得微信数据包并解析
    function response() {
      # 1. get well-formed XML string
      // $raw_data = $GLOBALS[ 'HTTP_RAW_POST_DATA' ];    // before php5.6
      
      $raw_data = file_get_contents('php://input');    // after php5.6
      
      # 2. transform xml to php object
      $data = simplexml_load_string($raw_data);
      
      switch (strtolower($data->MsgType)) {
        case 'event':
          responseEvent($data);
          break;
        case 'text':
          responseText($data);
          break;
        case 'image':
          responseNews($data);
          break;
        default: break;
      }
    }
```

## `access_token`

是公众号调用微信各个接口都需要用到的「全局、唯一票据」。或者说，**access_token 相当于一把钥匙，只有拥有这把钥匙才能获取微信的一些开放接口。**

### 与 `appid/appsecret` 的关系

`access_token` 由 `appid` 和 `appsecret` 生成，而 `appid` 和 `appsecret` 相当于是开发者在微信公众平台的唯一标识。

### 特点

- 唯一有效性
- 全局有效性
- 动态性

`access_token` 的值是动态的，有效时间为 2 小时，并以最新生成的为准。当access_token 过期之后，就需要通过 `appid` 和 `appsecret` 去重新调用微信公众平台的公共开放接口去重新生成。

当获取到新的 accees_token 的时候，旧的 access_token 的时候理论上已经 失效了，但是微信为了防止某些数据的丢失，在很短的时间内还是可以使用 旧的 access_token;但是尽量使用新的。

- 调用次数有限：2000次/天。

因此可以把它保存到 `Session`、`Memcache` 或者`数据库`中，而不用每次都去重新生成。

> 参考：手册 => 「中控服务器」。


# 常用 API

> API 最新变化以手册为准。
>
> 可以参考手册手写，也可以在项目中引入微信提供的 SDK（方法/函数的集合 可以形象地理解为装满了常用 方法/API/函数 的袋子），实现快速开发。如果是手写，也建议把自己写的逻辑封装为自己的 SDK，方便复用。
> 
> 调用所有微信开发接口时，均使用 `https` 协议。

## 用户关注/取消

> 此事件发生时，微信会把转发该事件到开发者填写的 URL，方便开发者给用户下发欢迎消息，或做账号的解绑。
    
```
    // 接受事件推送并响应
    function responseEvent($post_obj)
    {
        # 获得数据包中的数据
        # !!! 注意角色转换 这里的发送者和接收者是相对的
        $to_user     = $post_obj->FromUserName ;
        $from_user   = $post_obj->ToUserName ;
        
        # !!! 注意回复事件推送的 msg_type 也是文本, 不要写成 event 或直接 $post_obj->MsgType
        $msg_type    = 'text' ;
        $create_time = time() ;    // !!! 注意这里必须是整数
        
        # 判断是否是关注事件
        if (strtolower($post_obj->Event) == 'subscribe') {
            # 自定义回复内容
            $content     = <<< CTT
    		Aha ... 真厉害 ! 我都被你捉住啦 @_@
    		欢迎关注我的微信公众号 ( {$from_user} ) 哦 ^_^
    CTT;
        }
        
        # 指定保存回复内容的模板( 参考微信开发者手册, 注意内容的替换 )
        # !!! CDATA 节中的中括号不能多也不能少
        $tpl         = <<< TPL
    		<xml>
    		<ToUserName><![CDATA[%s]]></ToUserName>
    		<FromUserName><![CDATA[%s]]></FromUserName>
    		<CreateTime>%s</CreateTime>
    		<MsgType><![CDATA[%s]]></MsgType>
    		<Content><![CDATA[%s]]></Content>
    		</xml>
    TPL;
    
        # 使用 sprintf() 将内容替换并填充到模板
        # !!! 注意 sprintf() 中第一个参数后面的参数顺序必须和模板中的一致
        $info = sprintf($tpl, $to_user, $from_user, $create_time, $msg_type, $content) ;
        
        # 服务器本地调试
        // file_put_contents( 'response.txt' , $info."\n", FILE_APPEND ) ;
        
        echo $info ;
    }
```

## 回复文本消息

```
    // 回复文本消息
    function responseText($post_obj)
    {
        # 获得数据包中的数据
        # !!! 注意角色转换 这里的发送者和接收者是相对的
        $to_user     = $post_obj->FromUserName ;
        $from_user   = $post_obj->ToUserName ;
        $msg_type    = $post_obj->MsgType ;
        $create_time = time() ;    // !!! 注意这里必须是整数
        
        # 根据用户发送的文本消息作出不同的回复内容( 不区分大小写 )
        # 对用户输入的不同文本进行不同的处理
        
        $text = strtolower(trim($post_obj->Content)) ;
        switch ($text) {
            case '天气':
            case 'weather':
            case 'beijing':
            case '北京':
                responseWeather($post_obj, 'beijing') ;
                break ;
            case 'chongqing':
            case '重庆':
                responseWeather($post_obj, 'chongqing') ;
                break ;
            case 'shanghai':
            case '上海':
                responseWeather($post_obj, 'shanghai') ;
                break ;
            case 'guangzhou':
            case '广州':
                responseWeather($post_obj, 'guangzhou') ;
                break ;
            case 'shenzhen':
            case '深圳':
                responseWeather($post_obj, 'shenzhen') ;
                break ;
            case 'news':
                responseNews($post_obj) ;
                break ;
            case 'whoareu':
                $content = 'I am Chuanjiang Li, your best friends here. ^_^' . "\n\n" ;
                $content .= '我的个人主页是 : <a href="http://cjli.info">http://cjli.info</a> 欢迎浏览哦 @_@' ;
                break ;
            case 'whoami':
                $content = 'Your OpenID is : '.$to_user ;
                break ;
            case 'now':
                $content = '当前北京时间 ( CST ) :' . "\n\n" ;
                $content .= date('Y 年 m 月 d 日 H:i:s', time()) ;
                break ;
            case '我爱你':
            case 'I Love You':
                $content = '是吗 *_* 我更爱你哦~~~ $_$' ;
                break ;
            case 'SB':
            case 'sb':
            case 'wocao':
            case 'wc':
                $content = $text . ' 是什么 -_-' ;
                break ;
            case '笨蛋':
            case '傻逼':
                $content = $text . '骂谁呢 :(' ;
                break ;
            case '你真笨':
            case '你怎么这么笨':
            case '你好笨':
            case '笨死了你':
                $content = '555 ): 你才笨, 人家只是萌好嘛 -: -' ;
                break ;
            default:
                $content = '' ;
                if (is_numeric($text)) {
                    $content .= '你输入了一个数字, 嗯 ... 那 ' ;
                    $content .= $text . ' 是什么意思呢 ·_·' ;
                    break ;
                }
                $content .= '哎呀 ... 人家没听懂你的意思嘛 -_-# 换个说法呗 @_@' ;
                break ;
        }
        
        # 指定保存回复内容的模板( 参考微信开发者手册, 注意内容的替换 )
        # !!! CDATA 节中的中括号不能多也不能少
        $tpl         = <<< TPL
    		<xml>
    		<ToUserName><![CDATA[%s]]></ToUserName>
    		<FromUserName><![CDATA[%s]]></FromUserName>
    		<CreateTime>%s</CreateTime>
    		<MsgType><![CDATA[%s]]></MsgType>
    		<Content><![CDATA[%s]]></Content>
    		</xml>
    TPL;
    
        # 使用 sprintf() 将内容替换并填充到模板
        # !!! 注意 sprintf() 中第一个参数后面的参数顺序必须和模板中的一致
        $info = sprintf($tpl, $to_user, $from_user, $create_time, $msg_type, $content) ;
        
        # 服务器本地调试
        // file_put_contents( 'response.txt' , $info."\n", FILE_APPEND ) ;
        echo $info ;
    }
```

## 回复图文消息
    
```
    /**
     * 回复图文消息
     * !!! 注意 : 多条图文消息信息，默认第一个item为大图；如果图文数超过10，则将会无响应
     * 多图文的 description 不起作用; 单图文靠 title 来区分
     */
    function responseNews($post_obj)
    {
        $to_user   = $post_obj->FromUserName ;
        $from_user = $post_obj->ToUserName ;
        # 回复图文时 msg_type 必须是 news
        $msg_type  = 'news' ;
        $desc   = <<< CTT
        
    如果您看到了图文框架的输出表明本次测试成功
    如果没有看到图片, 可能是由于网络原因导致获取过慢或失败
    欢迎反馈 ! ^_^"
    
    CTT;
        $arr = array(
                array(
                    'title'       => '微信公众号 lamCJ-0 图文消息回复测试-1' ,
                    'description' => $desc ,
                    'pic_url'     => 'http://cjli.coding.me/images/default_avatar.jpg' ,
                    'url'         => 'https://mp.weixin.qq.com'
                ) ,
                array(
                    'title'       => '微信公众号 lamCJ-0 图文消息回复测试-2' ,
                    'description' => $desc ,
                    'pic_url'     => 'http://cjli.coding.me/images/default_avatar.jpg' ,
                    'url'         => 'https://mp.weixin.qq.com'
                ) ,
                array(
                    'title'       => '微信公众号 lamCJ-0 图文消息回复测试-3' ,
                    'description' => $desc ,
                    'pic_url'     => 'http://cjli.coding.me/images/default_avatar.jpg' ,
                    'url'         => 'https://mp.weixin.qq.com'
                )
        ) ;
        
        $tpl = '<xml>
    				<ToUserName><![CDATA[%s]]></ToUserName>
    				<FromUserName><![CDATA[%s]]></FromUserName>
    				<CreateTime>%s</CreateTime>
    				<MsgType><![CDATA[%s]]></MsgType>
    				<ArticleCount>'.count($arr).'</ArticleCount>
    				<Articles>' ;
        foreach ($arr as $k => $v) {
            $tpl .= '<item>
    				<Title><![CDATA['.$v['title'].']]></Title>
    				<Description><![CDATA['.$v['description'].']]></Description>
    				<PicUrl><![CDATA['.$v['pic_url'].']]></PicUrl>
    				<Url><![CDATA['.$v['url'].']]></Url>
    				</item>' ;
        }
        
        $tpl .= '</Articles>
    			</xml>' ;
    			
        $info = sprintf($tpl, $to_user, $from_user, time(), $msg_type) ;    // !!! 回复图文消息不需要 Content 参数
        echo $info ;
        // file_put_contents( 'image.log', $info, FILE_APPEND ) ;
        exit ;
    }
```

## 基础接口

### 获得 access_token

```
    /**
      * 通过 CURL 获得 access_token
      * access_token 相当于获得调用微信公众接口权限而必须的动态令牌
      * @return String access_token
      */
    function getAccessTokenByCurl()
    {
        # 1. 初始化 curl
        $ch        = curl_init() ;
        $appid     = APP_ID ;    // appid 和 appsecret 在 mp.weixin.qq.com 中获得
        $appsecret = APP_SECRET ;
        $url       = 'https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid='.$appid.'&secret='.$appsecret ;
        
        # 2. 设置 curl 的参数
        curl_setopt($ch, CURLOPT_URL, $url) ;
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1) ;    // 将 curl_exec() 获取的信息以文件流的形式返回, 而不是直接输出
        
        # 3. 采集
        $output = curl_exec($ch) ;
        
        # 4. 关闭 curl
        curl_close($ch) ;
        
        # 5. 数据处理
        if (curl_errno($ch)) {
            file_put_contents('access_token.log', curl_errno($ch)."\n", FILE_APPEND) ;
        }
        
        $arr = json_decode($output, true) ;
        return $arr[ 'access_token' ] ;
    }
```

### 获得微信服务器地址

```
    /**
      * 获取微信服务器 IP
      * 可以作为安全性验证, 判断数据来源是否是微信服务器
      * @param String $access_token
      * @return Array $arr
      */
    function getWXServerIPByCurl($access_token)
    {
        $url          = 'https://api.weixin.qq.com/cgi-bin/getcallbackip?access_token='.$access_token ;
        $ch           = curl_init() ;
        
        curl_setopt($ch, CURLOPT_URL, $url) ;
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1) ;
        
        $output       = curl_exec($ch) ;
        curl_close($ch) ;
        
        if (curl_errno($ch)) {
            file_put_contents('wxsercerip.log', curl_errno($ch)."\n", FILE_APPEND) ;
        }
        
        $arr          = json_decode($output, true) ;    // true 表示把 JSON 数据转换为 PHP 数组
        return $arr ;
    }
```


# 小实例

> 在微信公众号中也可以使用第三方的 API，比如：<u>http://apistore.baidu.com。</u>

## 天气查询

```
    /**
     * 天气查询响应
     * @param  [type] $post_obj [description]
     * @param  [type] $city     [description]
     * @return [type]           [description]
     */
    function responseWeather($post_obj, $city)
    {
        # 获得数据包中的数据
        # !!! 注意角色转换 这里的发送者和接收者是相对的
        $to_user     = $post_obj->FromUserName ;
        $from_user   = $post_obj->ToUserName ;
        $msg_type    = $post_obj->MsgType ;
        $create_time = time() ;    // !!! 注意这里必须是整数
        $ch = curl_init();
        $url = 'http://apis.baidu.com/heweather/weather/free?city='.$city ;
        $header = array(
            'apikey: 32_BIT_API_KEY',
        );
        
        // 添加 apikey到 header
        curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        
        // 执行HTTP请求
        curl_setopt($ch, CURLOPT_URL, $url);
        $res = curl_exec($ch);
        $arr = json_decode($res, true)['HeWeather data service 3.0'][0] ;
        print_r($arr) ;
        $cnty = $arr[ 'basic' ][ 'cnty' ] ;
        $city = $arr[ 'basic' ][ 'city' ] ;
        $id = $arr[ 'basic' ][ 'id' ] ;
        $update = $arr[ 'basic' ][ 'update' ][ 'loc' ] ;
        $lon = $arr[ 'basic' ][ 'lon' ] ;
        $lat = $arr[ 'basic' ][ 'lat' ] ;
        $status = $arr[ 'now' ][ 'cond' ][ 'txt' ] ;
        $tmp = $arr[ 'now' ][ 'tmp' ] ;
        $wind_deg = $arr[ 'now' ][ 'wind' ][ 'deg' ] ;
        $wind_dir = $arr[ 'now' ][ 'wind' ][ 'dir' ] ;
        $wind_sc = $arr[ 'now' ][ 'wind' ][ 'sc' ] ;
        $wind_spd = $arr[ 'now' ][ 'wind' ][ 'spd' ] ;
        $hum = $arr[ 'now' ][ 'hum' ] ;
        $vis = $arr[ 'now' ][ 'vis' ] ;
        echo '<hr>' ;
        $content = <<< CNTT
    	{$cnty}    {$city} ( 经度{$lon} 维度{$lat} )
    	{$status}    {$tmp} °C
    	{$wind_dir}{$wind_deg}°    {$wind_sc}级    {$wind_spd}kmph
    	相对湿度 {$hum}% ; 能见度 {$vis}km
    	更新时间 : {$update} CST ( {$id} )
    CNTT;
    
        # 指定保存回复内容的模板( 参考微信开发者手册, 注意内容的替换 )
        # !!! CDATA 节中的中括号不能多也不能少
        $tpl         = <<< TPL
    		<xml>
    		<ToUserName><![CDATA[%s]]></ToUserName>
    		<FromUserName><![CDATA[%s]]></FromUserName>
    		<CreateTime>%s</CreateTime>
    		<MsgType><![CDATA[%s]]></MsgType>
    		<Content><![CDATA[%s]]></Content>
    		</xml>
    TPL;
    
        # 使用 sprintf() 将内容替换并填充到模板
        # !!! 注意 sprintf() 中第一个参数后面的参数顺序必须和模板中的一致
        $info = sprintf($tpl, $to_user, $from_user, $create_time, $msg_type, $content) ;
        
        # 服务器本地调试
        // file_put_contents( 'weather_data.txt' , $arr."\n", FILE_APPEND ) ;
        echo $info ;
    }
```

# FAQ

## 如何获取更多接口？

> 1、可以在公众平台网站中申请微信认证，认证成功后，将获得更多接口权限，满足更多业务需求。
> 
> 2、微信认证暂不支持个人类型的公众帐号申请微信认证。
> 
> 3、微信公众号认证需要 300 RMB / 次的手续费。

## Web 开发时微信支付开发过程很繁琐？

强烈建议参考的手册优先参考 [微信支付文档](https://pay.weixin.qq.com/wiki/doc/api/index.html)，而不是 [微信公众平台技术文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1445241432) 中那个 JS-SDK 说明文档。


# 参考

- [网页授权获取用户基本信息](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1445241432)
- [微信商户平台-微信支付 SDK 下载](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=11_1)
- [微信公众平台-JSSDK 下载](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1445241432)



