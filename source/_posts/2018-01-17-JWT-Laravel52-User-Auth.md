---
title: JWT 多用户认证
date: 2018-01-17 11:04:43
categories: Laravel
tags: [JWT, Laravel, Framework]
---

## Json Web Token
> JWT在Laravel5.2中的应用

JWT代表Json Web Token.JWT能有效地进行身份验证并连接前后端。

 - 降地耦合性，取代session，进一步实现前后端分离
 - 减少服务器的压力
 - 可以很简单的实现单点登录
 
<!--more-->

## 实现API

### 安装扩展

```
    composer require tymon/jwt-auth
```

之后打开config/app.php文件添加service provider 和 aliase

### config/app.php

```
    'providers' => [
        ....
        Tymon\JWTAuth\Providers\JWTAuthServiceProvider::class,       // 注意这里的名字,下文会提到
    ],
    'aliases' => [
        ....
        'JWTAuth' => Tymon\JWTAuth\Facades\JWTAuth::class
    ],
```

OK，现在来发布JWT的配置文件，比如令牌到期时间配置等
```
    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\JWTAuthServiceProvider"
```

最后一步需要生成JWT Key
```
    php artisan jwt:generate
```

### 创建API路由

我在创建Api路由的时候会用到一个`“cors”`中间件，虽然它不是强制性的，但是后面你会发现报类似这样的错

```
    Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://xxx.com/api/register. (Reason: CORS header 'Access-Control-Allow-Origin' missing)
```

大致翻译下，“跨源请求阻塞:同源策略不允许读取http://kylesean.com/api/register远程资源。(原因:CORS 头“Access-Control-Allow-Origin” 没有)。” 这就是跨域请求导致的错误消息，当然你可以自定义Header，Origin, Method来解决跨域问题,不过我这边推荐一个package：barryvdh/laravel-cors(最新稳定版是0.8.2)，这里安装过程省略。


### 创建中间件

```
    php artisan make:middleware CORS
```

进入app/Http/Middleware，编辑CORS.php

### app/Http/Middleware/CORS.php
```
    namespace App\Http\Middleware;
    use Closure;
    class CORS
    {
        public function handle($request, Closure $next)
        {
            header('Access-Control-Allow-Origin: *');
    
            $headers = [
                'Access-Control-Allow-Methods'=> 'POST, GET, OPTIONS, PUT, DELETE',
                'Access-Control-Allow-Headers'=> 'Content-Type, X-Auth-Token, Origin'
            ];
            if($request->getMethod() == "OPTIONS") {
                return Response::make('OK', 200, $headers);
            }
    
            $response = $next($request);
            foreach($headers as $key => $value)
                $response->header($key, $value);
            return $response;
        }
    }
```

Ok,在app/Http/Kernel.php注册中间件

### app/Http/Kernel.php

```
    namespace App\Http;
    use Illuminate\Foundation\Http\Kernel as HttpKernel;
    class Kernel extends HttpKernel
    {
        ...
        ...
        protected $routeMiddleware = [
            ...
            'cors' => \App\Http\Middleware\CORS::class,
        ];
    }
```

有了这个中间件我们就解决了跨域问题。接下来回到路由

### app/Http/routes.php

```
    Route::group(['middleware' => ['api','cors'],'prefix' => 'api'], function () {
        Route::post('register', 'ApiController@register');     // 注册
        Route::post('login', 'ApiController@login');           // 登陆
        Route::group(['middleware' => 'jwt.auth'], function () {
            Route::post('get_user_details', 'APIController@get_user_details');  // 获取用户详情
        });
    });
```

> 建议：过滤掉路由api/*下的`csrf_token`，方便测试开发

上面的jwt-auth中间件现在还是无效的，接着创建这个middleware
```
    php artisan make:middleware authJWT
```

同样的我们需要编辑下这个authJWT.php

### app/Http/Middleware/authJWT.php

```
    namespace App\Http\Middleware;
    use Closure;
    use Tymon\JWTAuth\Facades\JWTAuth;
    use Exception;
    class authJWT
    {
        public function handle($request, Closure $next)
        {
            try {
                // 如果用户登陆后的所有请求没有jwt的token抛出异常
                $user = JWTAuth::toUser($request->input('token')); 
            } catch (Exception $e) {
                if ($e instanceof \Tymon\JWTAuth\Exceptions\TokenInvalidException){
                    return response()->json(['error'=>'Token 无效']);
                }else if ($e instanceof \Tymon\JWTAuth\Exceptions\TokenExpiredException){
                    return response()->json(['error'=>'Token 已过期']);
                }else{
                    return response()->json(['error'=>'出错了']);
                }
            }
            return $next($request);
        }
    }
```

OK，接着注册该中间件

### app/Http/Kernel.php
```
    namespace App\Http;
    use Illuminate\Foundation\Http\Kernel as HttpKernel;
    class Kernel extends HttpKernel
    {
        ...
        ...
        protected $routeMiddleware = [
            ...
            'jwt.auth' => \App\Http\Middleware\authJWT::class,
        ];
    }
```

然后，我们创建控制器管理所有的请求

### app/Http/Controllers/ApiController.php
```
    <?php
    
    namespace App\Http\Controllers;
    use Illuminate\Http\Request;
    use App\User;
    use Illuminate\Support\Facades\Hash;
    use Tymon\JWTAuth\Facades\JWTAuth;
    
    class ApiController extends Controller
    {
        /*注册*/
        public function register(Request $request)
        {
            $input = $request->all();
            $input['password'] = Hash::make($input['password']);
            User::create($input);
            return response()->json(['result'=>true]);
        }
    
        /*登陆*/
        public function login(Request $request)
        {
            $input = $request->all();
            if (!$token = JWTAuth::attempt($input)) {
                return response()->json(['result' => '邮箱或密码错误.']);
            }
            return response()->json(['result' => $token]);
        }
    
        /*获取用户信息*/
        public function get_user_details(Request $request)
        {
            $input = $request->all();
            $user = JWTAuth::toUser($input['token']);
            return response()->json(['result' => $user]);
        }
    
    }
```


最后一步我们就来模拟一个请求来测试这个api,为了模拟本地跨域请求，我们简单的新建一个静态页面test.html
```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Document</title>
        <script src="http://apps.bdimg.com/libs/jquery/2.1.1/jquery.min.js"></script>
    </head>
    <body>
    
    </body>
    <script>
    $.ajax({
        url: "http://localhost/api/login",
        dataType: "json",
        type: "POST",
        data: {"email":kylesean@qq.com","password":"123456"},
        success: function (data) {
            alert(data.result)
        }
        // 这里我们用ajax请求测试，当然你也可以用Angular.js  Vue.js
    });
    </script>
    </html>
```

这里我们要注意一下，以上测试我们仍是基于User table的,我们来模拟一下login过程，如果账号密码匹配成功，不出意外将会出现类似：
```
    {
      "result": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOjEsImlzcyI6Imh0dHA6XC9cL2xvY2FsaG9zdFwvYXBpXC9sb2dpbiIsImlhdCI6MTQ3MzQ1MjUyNSwiZXhwIjoxNDczNDU2MTI1LCJuYmYiOjE0NzM0NTI1MjUsImp0aSI6IjA1M2IzNjliYzYyZjJiZjJmMGMxNjFiNzIxNzY4Y2MzIn0.4WeezpSgEKjNmDFxv1nMU9HxqJgBE7bPyaJDRK4iLeA"
    }
```

> 至此，我们已经实现了jwt的认证功能，那么我们接着完成下一半工作，实现jwt的多用户认证，即`Jwt for Multi Auth.`
如果你的业务场景是的确需要多用户认证，比如为管理员admin单独生成一张表，恰好字段也是laravel auth user里面默认的name email password remember_token等，那么实现起来就方便的多，官方文档和网上的demo示例已经很多了，但是若结合这个laravel/jwt-auth扩展进行多用户认证，其实坑还是蛮多的，由于该扩展0.5.9似乎不支持多用户认证(反正不会帮我们自定义好guard，当然我们可以自己在AuthServiceProvider里用boot方法实现) 我在`github issue`里面看到好多人踩过此坑，结合我遇到的

>总结一下，里面一个哥们说，得用^0.1@dev版本(什么鬼，what's the fuck!),so 继续撸之：

## 实现JWT的多用户认证

### composer.json里修改为
```
    "require": {
        ...
        "tymon/jwt-auth": "^1.0@dev",  // 修改之前的,Or making a fresh start 
        ...
    }
```

### 同样app.php里进行配置
```
    'providers' => [
        ....
        Tymon\JWTAuth\Providers\LaravelServiceProvider::class,  // 上文已经提到过，这里的provider已经不是JWTauthServiceProvider
    ],
    'aliases' => [
        ....
        'JWTAuth' => Tymon\JWTAuth\Facades\JWTAuth::class
    ],
```

### 发布配置文件
```
    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
```

### 生成密钥
```
    php artisan jwt:secret   // 发现没 生成key的方法也变了，不是 php artisan jwt:generate了
```

接下来就是重点了，要设置好config/auth.php里面的配置项了，这里不能乱设置：

### config/auth.php
```
    /**
     * 默认使用web这个guard
     */
    'defaults' => [
            'guard' => 'web',
            'passwords' => 'users',
        ],
    
    'guards' => [
            'web' => [
                'driver' => 'session',
                'provider' => 'users',
            ],
    
            'api' => [
                'driver' => 'token',
                'provider' => 'users',
            ],
            /**
             * 这里是我自定义的guard,这里我叫staffs，你也可以根据自己的业务需求设置admins等，并且我
             * 需要实现json web token认证
             */
            'staffs' => [
                'driver'   => 'jwt',   // 结合扩展这里定义即生效
                'provider' => 'staffs'
            ]
    
        ],
    
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,   // 这里注意修改命名空间 通常是'model' => App\Models\User::class,
        ],
    
        /**
         * 同样的这里定义自己的provider
         */
        'staffs' => [
            'driver' => 'eloquent',
            'model' => App\Models\Staff::class,
        ]
    
        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],
    
    'passwords' => [
        'users' => [
            'provider' => 'users',
            'email' => 'auth.emails.password',
            'table' => 'password_resets',
            'expire' => 60,
        ],
    
        /**
         * 这里我并没有设置如下，因为我的staff表并没有email字段，默认的重置密码功能暂时没考虑
         */
       <!-- 'staffs' => [
            'provider' => 'staffs',
            'email' => 'auth.emails.password',
            'table' => 'password_resets',
            'expire' => 60,                      
        ],-->
    ]
```


下一步，创建我们的staff model
### Models\staff.php

```
    <?php
    
    namespace App\Models;
    
    use Illuminate\Auth\Authenticatable;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Auth\Passwords\CanResetPassword;
    use Illuminate\Foundation\Auth\Access\Authorizable;
    use Illuminate\Contracts\Auth\Authenticatable as AuthenticatableContract;
    use Illuminate\Contracts\Auth\Access\Authorizable as AuthorizableContract;
    use Tymon\JWTAuth\Contracts\JWTSubject as AuthenticatableUserContract;  
    
    class Staff extends Model implements AuthenticatableContract, AuthorizableContract,  AuthenticatableUserContract
    {
        use Authenticatable, Authorizable, CanResetPassword;
    
        protected $table = 'staffs';
        protected $fillable = ['name', 'phone', 'password'];
        protected $hidden = ['password', 'remember_token'];
    
        public function getJWTIdentifier()
        {
            return $this->getKey(); // Eloquent model method
        }
    
        /**
         * @return array
         */
        public function getJWTCustomClaims()
        {
            return [];
        }
    }
```

好吧，接下来我们又要添加相关路由了

```
    Route::post('/api/login', 'StaffAuthController@login');
    Route::post('/api/register', 'StaffAuthController@register');
```

控制器书写我们的业务逻辑
```
    namespace App\Http\Controllers\Staff;
    
    use App\Models\Staff;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    use App\Http\Requests;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;
    use Illuminate\Support\Facades\Validator;
    use Tymon\JWTAuth\Facades\JWTAuth;
    use Illuminate\Support\Facades\Auth;
    
    class StaffAuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;
        protected $guard = 'staffs';
    
        /*注册*/
        public function register(Request $request)
        {
            $this->validate($request, [
                'phone' => 'required|max:16',
                'password' => 'required|min:6',
            ]);
            $credentials = [
                'phone' => $request->input('phone'),
                'password' => bcrypt($request->input('password')),
            ];
    
            $id = Staff::create($credentials);
            if ($id) {
                $token = Auth::guard($this->getGuard())->attempt($credentials); // 也可以直接guard('staffs')
                return response()->json(['result' => $token]);
            }
        }
    
        /*登录*/
        public function login(Request $request)
        {
    
            $credentials = $request->only('phone','password');
            if ( $token = Auth::guard($this->getGuard())->attempt($credentials) ) {
    
                return response()->json(['result' => $token]);
            } else {
                return response()->json(['result'=>false]);
            }
        }
    {
```

到现在，一个基于JWT的多用于认证系统雏形就建立就来了，这里面需要完善的东西很多，比如刷新token,退出登录，增加额外的中间件等，

## 参考

- [Laravel 5.2 使用 JWT 完成多用户认证](https://laravel-china.org/topics/2811/laravel-52-uses-jwt-to-complete-multi-user-authentication)
- [Feature: Laravel 5.2 Custom Authentication Guard and Driver](https://github.com/tymondesigns/jwt-auth/issues/513)
