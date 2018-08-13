---
title: 实名认证 「聚合数据」
date: 2018-08-01 14:54:28
categories: ThirdParty
tags: [第三方, 实名认证]
---

> 实名认证牵涉公安备案系统，需向公安机关申请接口进行验证。当然，调用其他已向公安机关申请了接口的第三方接口一样可行

<!-- more -->

# 聚合数据

> [身份证实名认证文档](https://www.juhe.cn/docs/api/id/103)

## API文档

- 接口地址:

```
    http://op.juhe.cn/idcard/query
```

- 返回格式:

```
    json
```

- 请求方式:

```
    http get/post
```

- 请求示例:

```
    http://op.juhe.cn/idcard/query?key=您申请的KEY&idcard=420104198905015713&realname=%E7%8E%8B%E9%9D%9E%E5%90%9F
```

- 接口备注:

```
    error_code 为0时计费
```

# 直接上源码

```php
    <?php
    
    //----------------------------------
    // 身份证实名认证 － 聚合数据
    // 在线接口文档：http://www.juhe.cn/docs/103
    //----------------------------------
    
    header('Content-type:text/html;charset=utf-8');
    
    //配置您申请的appkey
    $appkey = "*********************";
    
    //************1.真实姓名和身份证号码判断是否一致************
    $url = "http://op.juhe.cn/idcard/query";
    
    $params = array(
        "idcard"   => "36072219940809****",  //身份证号码
        "realname" => "曹贤亮",               //真实姓名
        "key"      => $appkey,               //应用APPKEY(应用详细页查询)
    );
    
    $paramstring = http_build_query($params);
    $content = juhecurl($url,$paramstring);
    $result = json_decode($content,true);
    
    
    if($result){
        if($result['error_code'] == '0'){
            if($result['result']['res'] == '1'){
                echo "身份证号码和真实姓名一致";
            } else {
                echo "身份证号码和真实姓名不一致";
            }
        } else {
            echo $result['error_code'].":".$result['reason'];
        }
    } else {
        echo "请求失败";
    }
    //**************************************************
    
    
    /**
     * 请求接口返回内容
     * @param  string $url [请求的URL地址]
     * @param  string $params [请求的参数]
     * @param  int $ipost [是否采用POST形式]
     * @return  string
     */
    function juhecurl($url, $params = false, $ispost = 0){
        $httpInfo = array();
        $ch = curl_init();
    
        curl_setopt( $ch, CURLOPT_HTTP_VERSION , CURL_HTTP_VERSION_1_1 );
        curl_setopt( $ch, CURLOPT_USERAGENT , 'JuheData' );
        curl_setopt( $ch, CURLOPT_CONNECTTIMEOUT , 60 );
        curl_setopt( $ch, CURLOPT_TIMEOUT , 60);
        curl_setopt( $ch, CURLOPT_RETURNTRANSFER , true );
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
    
        if( $ispost ) {
            curl_setopt( $ch , CURLOPT_POST , true );
            curl_setopt( $ch , CURLOPT_POSTFIELDS , $params );
            curl_setopt( $ch , CURLOPT_URL , $url );
        } else {
            if($params) {
                curl_setopt( $ch , CURLOPT_URL , $url.'?'.$params );
            } else {
                curl_setopt( $ch , CURLOPT_URL , $url);
            }
        }
    
        $response = curl_exec( $ch );
    
        if ($response === FALSE) {
            //echo "cURL Error: " . curl_error($ch);
            return false;
        }
    
        $httpCode = curl_getinfo( $ch , CURLINFO_HTTP_CODE );
        $httpInfo = array_merge( $httpInfo , curl_getinfo( $ch ) );
    
        curl_close( $ch );
    
        return $response;
    }
```

# 阿里云的实名认证

## API文档

> [身份证实名认证](https://market.aliyun.com/products/56928004/cmapi014760.html?spm=5176.730005.0.0.aeWVBT#sku=yuncode876000009)

- 调用地址:

```
    http://idcard.market.alicloudapi.com/lianzhuo/idcard
```

- 请求方式:

```
    GET
```

- 返回类型:

```
    JSON
```

## `PHP`请求示例

```
    <?php
    
        $host = "http://idcard.market.alicloudapi.com";
        $path = "/lianzhuo/idcard";
        $method = "GET";
        $appcode = "你自己的AppCode";
        $headers = array();
        array_push($headers, "Authorization:APPCODE " . $appcode);
        $querys = "cardno=370703198111300338&name=%E9%83%AD%E5%BE%B7%E6%98%8C";
        $bodys = "";
        $url = $host . $path . "?" . $querys;
    
        $curl = curl_init();
        curl_setopt($curl, CURLOPT_CUSTOMREQUEST, $method);
        curl_setopt($curl, CURLOPT_URL, $url);
        curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($curl, CURLOPT_FAILONERROR, false);
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($curl, CURLOPT_HEADER, true);
        if (1 == strpos("$".$host, "https://"))
        {
            curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
            curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false);
        }
        var_dump(curl_exec($curl));
    ?>
```