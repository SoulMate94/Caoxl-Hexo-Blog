---
title: 基于Lumen-CORS跨域
date: 2018-08-10 09:18:27
categories: Lumen
tags: [Lumen, CORS]
---

> 如果是客户端是浏览器，则在调用 API 的时候，基本上无法避免跨域问题。
> 不过处理也很简单，需要浏览器和服务器同时配合：浏览器在发起 `OPTIONS preflight` 请求的时候，服务器返回认可的一些信息就行了。

<!-- more -->

# 什么是跨域 ?

浏览器的同源策略会导致跨域，这里同源策略又分为以下两种:

- **DOM同源策略**：禁止对不同源页面DOM进行操作。这里主要场景是`iframe`跨域的情况，不同域名的`iframe`是限制互相访问的。
- **XmlHttpRequest同源策略**：禁止使用`XHR`对象向不同源的服务器地址发起`HTTP`请求。

> 只要协议、域名、端口有任何一个不同，都被当作是不同的域，之间的请求就是跨域操作。

# 为什么要有跨域限制

了解完跨域之后，想必大家都会有这么一个思考，为什么要有跨域的限制，浏览器这么做是出于何种原因呢。其实仔细想一想就会明白，**跨域限制主要是为了安全考虑**。

- **AJAX同源策略主要用来防止`CSRF`攻击**。如果没有`AJAX`同源策略，相当危险，我们发起的每一次`HTTP`请求都会带上请求地址对应的`cookie`，那么可以做如下攻击：

1. 用户登录了自己的银行页面 `http://mybank.com，http://mybank.com`向用户的`cookie`中添加用户标识。
2. 用户浏览了恶意页面 `http://evil.com`。执行了页面中的恶意AJAX请求代码。
3. `evil.com`向`http://mybank.com`发起`AJAX HTTP`请求，请求会默认把`http://mybank.com`对应`cookie`也同时发送过去。
4. 银行页面从发送的`cookie`中提取用户标识，验证用户无误，`response`中返回请求数据。此时数据就泄露了。
5. 而且由于`Ajax`在后台执行，用户无法感知这一过程。


- **DOM同源策略**也一样，如果`iframe`之间可以跨域访问，可以这样攻击：

1. 做一个假网站，里面用`iframe`嵌套一个银行网站 `http://mybank.com`。
2. 把`iframe`宽高啥的调整到页面全部，这样用户进来除了域名，别的部分和银行的网站没有任何差别。
3. 这时如果用户输入账号密码，我们的主网站可以跨域访问到`http://mybank.com`的`dom`节点，就可以拿到用户的输入了，那么就完成了一次攻击。

所以说有了跨域跨域限制之后，我们才能更安全的上网了。

# 如何解决跨域 ?

## 跨域资源共享 - CORS

> `CORS`是一个W3C标准，全称是”跨域资源共享”（`Cross-origin resource sharing`）

具体详见下面的代码封装~~~

## `Jsonp`实现跨域

基本原理就是通过动态创建`script`标签,然后利用`src`属性进行跨域。

这么说比较模糊，我们来看个例子:

```
    // 定义一个fun函数
    function fun(fata) {
        console.log(data);
    };
    // 创建一个脚本，并且告诉后端回调函数名叫fun
    var body = document.getElementsByTagName('body')[0];
    var script = document.gerElement('script');
    script.type = 'text/javasctipt';
    script.src = 'demo.js?callback=fun';
    body.appendChild(script);
```

返回的js脚本，直接会执行。所以就执行了事先定义好的fun函数了，并且把数据传入了进来。

```
    fun({"name": "name"})
```

当然，这个只是一个原理演示，实际情况下，**我们需要动态创建这个fun函数，并且在数据返回的时候销毁它**。

因为在实际使用的时候，我们用的各种`ajax`库，基本都包含了`jsonp`的封装，不过我们还是要知道一下原理，不然就不知道为什么`jsonp`不能发`post`请求了~

## 服务器代理

浏览器有跨域限制，但是服务器不存在跨域问题，所以可以由服务器请求所要域的资源再返回给客户端。

> 服务器代理是万能的。

### `document.domain`来跨子域

### `location.hash`跨域

### 使用`postMessage`实现页面之间通信

# 基于`Lumen`封装`CORS`

具体如下:

- `app/Http/Middleware/CORS.php`

```php
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

- `bootstrap/app.php`

```
    // 指定全局中间件
    $app->middleware([
       'cors' => App\Http\Middleware\CORS::class,
    ]);
```

- `jQuery Ajax Demo`

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
