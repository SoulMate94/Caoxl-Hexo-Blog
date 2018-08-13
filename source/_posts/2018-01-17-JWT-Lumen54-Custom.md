---
title: JWT-Lumen5.4-自定义
date: 2018-01-17 11:44:58
categories: Lumen
tags: [Lumen, JWT, Framework]
---

## JWT在Lumen5.4中的应用

> 在公司Lumen5.4框架中使用了JWT
> - Lumen5.4开发记录另见: [Lumen5.4-开发记录-转](http://blog.caoxl.com/2018/01/05/Lumen54-%E5%BC%80%E5%8F%91%E8%AE%B0%E5%BD%95-%E8%BD%AC/)


<!--more-->


## 启用自定义 JWT 认证

 - composer.json中添加(推荐composer安装)
 ```
    "require": {
        "firebase/php-jwt": "^5.0",
    }
 ```

- 首先在 bootstrap/app.php 中启用：

```
    $app->routeMiddleware([
        'auth_jwt' => App\Http\Middleware\Auth\JWT::class,
    ]);
```

- 然后修改 app/Http/Middleware/Auth/JWT.php 为：

```
    <?php
    namespace App\Http\Middleware\Auth;
    use Closure;
    use Firebase\JWT\JWT as FirebaseJWT;
    class JWT
    {
        protected $auth = false;
        public function __construct()
        {
            try {
                if (isset($_SERVER['HTTP_AUTHORIZATION'])) {
                    $this->auth = FirebaseJWT::decode(
                        $_SERVER['HTTP_AUTHORIZATION'],
                        env('APP_KEY'),
                        ['HS256']
                    );
                }
            } catch (\Exception $e) {
            } finally {
            }
        }
        public function handle($request, Closure $next)
        {
            if (false === $this->auth) {
                return response()->json([
                    'error' => 'Unauthorized'
                ], 401);
            }
            $request->attributes->add([
                'auth_jwt' => $this->auth,
            ]);
            return $next($request);
        }
    }
```


## 将中间件的值传递给控制器
> Pass value from middleware to controller

```
    / 1. Middleware
    public function handle($request, Closure $next)
    {
        $request->attributes->add([
          	'key' => 'value'
     	]);
      
      	return $next($request);
    }
    // 2. Controller
    public function getVauleFromMiddleware(Request $req)
    {
      	echo $req->get('key');    // output: value
    }
```

通过这种方式，可以对不同的路由使用不同的认证（非常语义化），然后把各种认证的结果（jwt，七牛）传递到控制器，可以使控制器不用再操心认证逻辑，减少控制器代码量。

## CORS

> CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

如果是客户端是浏览器，则在调用 API 的时候，基本上无法避免跨域问题。

不过处理也很简单，需要浏览器和服务器同时配合：浏览器在发起 OPTIONS preflight 请求的时候，服务器返回认可的一些信息就行了。

**具体如下：**
 - app/Http/Middleware/CORS.php
 ```
    <?php
    namespace App\Http\Middleware;
    use Closure;
    class CORS
    {
    	public function __construct()
    	{
    	}
    	public function handle($request, Closure $next)
    	{
            $headers = [
                'Access-Control-Allow-Origin'  => '*',
                'Access-Control-Allow-Methods' => '*',
                'Access-Control-Allow-Headers' => 'Access-Control-Allow-Origin, AUTHORIZATION',
                'Access-Control-Max-Age'       => 86400,
            ];
            if ('OPTIONS' == $request->getMethod()) {
            	return response(null, 200, $headers);
            }
            return $next($request)->withHeaders($headers);
    	}
    }
 ```
 
- bootstrap/app.php
```
    // 指定全局中间件
    $app->middleware([
       'cors' => App\Http\Middleware\CORS::class,
    ]);
```

- jQuery AJAX Demo
```
    $.ajax({
      url: 'http://api.example.com/path/to/resource',
      type: 'GET',
      dataType: 'json',
      async: false,
      cors: true ,
      headers: {
        'Access-Control-Allow-Origin': '*',
        'AUTHORIZATION': 'JWT_STRING'
      },
      success: function (res) {
      },
      error: function (xhr, status) {
      }
    });
```

## Example

```php
    <?php
    use \Firebase\JWT\JWT;
    
    $key = "example_key";
    $token = array(
        "iss" => "http://example.org",
        "aud" => "http://example.com",
        "iat" => 1356999524,
        "nbf" => 1357000000
    );
    
    $jwt = JWT::encode($token, $key);
    $decoded = JWT::decode($jwt, $key, array('HS256'));
    
    print_r($decoded);
    
    
    $decoded_array = (array) $decoded;
    
    JWT::$leeway = 60; // $leeway in seconds
    $decoded = JWT::decode($jwt, $key, array('HS256'));
    
    ?>
```

## Example with RS256 (openssl)

```php
    <?php
    use \Firebase\JWT\JWT;
    
    $privateKey = <<<EOD
    -----BEGIN RSA PRIVATE KEY-----
    MIICXAIBAAKBgQC8kGa1pSjbSYZVebtTRBLxBz5H4i2p/llLCrEeQhta5kaQu/Rn
    vuER4W8oDH3+3iuIYW4VQAzyqFpwuzjkDI+17t5t0tyazyZ8JXw+KgXTxldMPEL9
    5+qVhgXvwtihXC1c5oGbRlEDvDF6Sa53rcFVsYJ4ehde/zUxo6UvS7UrBQIDAQAB
    AoGAb/MXV46XxCFRxNuB8LyAtmLDgi/xRnTAlMHjSACddwkyKem8//8eZtw9fzxz
    bWZ/1/doQOuHBGYZU8aDzzj59FZ78dyzNFoF91hbvZKkg+6wGyd/LrGVEB+Xre0J
    Nil0GReM2AHDNZUYRv+HYJPIOrB0CRczLQsgFJ8K6aAD6F0CQQDzbpjYdx10qgK1
    cP59UHiHjPZYC0loEsk7s+hUmT3QHerAQJMZWC11Qrn2N+ybwwNblDKv+s5qgMQ5
    5tNoQ9IfAkEAxkyffU6ythpg/H0Ixe1I2rd0GbF05biIzO/i77Det3n4YsJVlDck
    ZkcvY3SK2iRIL4c9yY6hlIhs+K9wXTtGWwJBAO9Dskl48mO7woPR9uD22jDpNSwe
    k90OMepTjzSvlhjbfuPN1IdhqvSJTDychRwn1kIJ7LQZgQ8fVz9OCFZ/6qMCQGOb
    qaGwHmUK6xzpUbbacnYrIM6nLSkXgOAwv7XXCojvY614ILTK3iXiLBOxPu5Eu13k
    eUz9sHyD6vkgZzjtxXECQAkp4Xerf5TGfQXGXhxIX52yH+N2LtujCdkQZjXAsGdm
    B2zNzvrlgRmgBrklMTrMYgm1NPcW+bRLGcwgW2PTvNM=
    -----END RSA PRIVATE KEY-----
EOD;
    
    $publicKey = <<<EOD
    -----BEGIN PUBLIC KEY-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC8kGa1pSjbSYZVebtTRBLxBz5H
    4i2p/llLCrEeQhta5kaQu/RnvuER4W8oDH3+3iuIYW4VQAzyqFpwuzjkDI+17t5t
    0tyazyZ8JXw+KgXTxldMPEL95+qVhgXvwtihXC1c5oGbRlEDvDF6Sa53rcFVsYJ4
    ehde/zUxo6UvS7UrBQIDAQAB
    -----END PUBLIC KEY-----
EOD;
    
    $token = array(
        "iss" => "example.org",
        "aud" => "example.com",
        "iat" => 1356999524,
        "nbf" => 1357000000
    );
    
    $jwt = JWT::encode($token, $privateKey, 'RS256');
    echo "Encode:\n" . print_r($jwt, true) . "\n";
    
    $decoded = JWT::decode($jwt, $publicKey, array('RS256'));
    $decoded_array = (array) $decoded;
    echo "Decode:\n" . print_r($decoded_array, true) . "\n";
?>
```

## Lumen5.4-Example

