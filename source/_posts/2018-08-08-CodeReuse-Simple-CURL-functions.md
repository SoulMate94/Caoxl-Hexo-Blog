---
title: 「代码复用」 CURL
date: 2018-08-08 10:22:00
categories: CodeReuse
tags: [代码复用, CURL]
---

> Don't repeat yourself

<!-- more -->

# 直接上源码

```php
    <?php
    
    // Simple CURL Helper funcitons
    // @caoxl
    
    namespace App\Traits;
    
    trait CURL
    {
        public function requestJsonApi(
            $uri,
            $type = 'POST',
            $params = [],
            $headers = []
        ) {
            $headers = [
                'Content-Type: application/json; Charset=UTF-8',
            ];
    
            $res = $this->requestHTTPApi($uri, $type, $headers, $params);
    
            if (! $res['err']) {
                $res['res'] = json_decode($res['res'], true);
            }
    
            return $res;
        }
    
        public function requestHTTPApi(
            string $uri,
            string $type = 'GET',
            array $headers = [],
            $data
        ) {
            $setOpt = [
                CURLOPT_URL            => $uri,
                CURLOPT_RETURNTRANSFER => true,
            ];
    
            if ($headers) {
                $setOpt[CURLOPT_HTTPHEADER] = $headers;
            }
    
            if ('POST' == $type) {
                $setOpt[CURLOPT_POST]       = true;
                $setOpt[CURLOPT_POSTFIELDS] = $data;
            }
    
            $ch = curl_init();
            curl_setopt_array($ch, $setOpt);
            $res = curl_exec($ch);
    
            $errNo  = curl_errno($ch);
            $errMsg = curl_error($ch);
    
            curl_close($ch);
    
            return [
                'err' => $errNo,
                'msg' => ($errMsg ?: 'ok'),
                'res' => $res,
            ];
        }
    }
```

# 说明

- **`Traits`**

> https://secure.php.net/manual/zh/language.oop5.traits.php
> 自 PHP 5.4.0 起，PHP 实现了一种代码复用的方法，称为 trait。...

- **`requestHTTPApi`**

  - `curl_init()` - 初始化新的会话，返回 cURL 句柄，供`curl_setopt()`、 `curl_exec()` 和 `curl_close()` 函数使用。
  - `curl_setopt_array()` - 为 cURL 传输会话批量设置选项。这个函数对于需要设置大量的 cURL 选项是非常有用的，不需要重复地调用 `curl_setopt()`。
  - `curl_setopt()` - 设置 cURL 传输选项
  > `curl_setopt($ch, $option, $value)`
  > 点击查看可以设置的<u>[CURLOPT_XXX选项](https://secure.php.net/manual/zh/function.curl-setopt.php)</u>
  - `curl_exec()` - 执行给定的 cURL 会话。
  - `curl_errno()` - 返回最后一次 cURL 操作的错误代码。
  - `curl_error()` - 返回最近一次 cURL 操作的文本错误详情。
  - `curl_close()` - 关闭 cURL 会话并且释放所有资源。cURL 句柄 ch 也会被删除。

- **`requestJsonApi`**

  - `json_decode()` - 对 JSON 格式的字符串进行解码, 当该参数为 TRUE 时，将返回 array 而非 object 。