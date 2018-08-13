---
title: Laravel 5.5 with DA - LA 
date: 2018-01-31 11:42:34
categories: Laravel
tags: [Laravel, DingoAPI, LaravelAdmin, Framework]
---

本文主要总结使用 Laravel 5.5，及配套的一些组件在进行开发遇到的一些值得记录的东西，以及坑。

> Laravel 5.5 - Dingo API - Laravel admin 

<!--more-->

# Laravel 5.5

## 重拾 Tinker

这货真的很好用，Laravel 引入的所有类有很多方法名我们其实是不知道的，但是在 tinker 中可以被全部补全出来。

当然除了补全 Laravel 中的类方法和全局函数，也可以和 php -a 一样补全 PHP 内置的函数，且能少些不少语法。

总之，对开发期间帮助性很强，十分推荐。

## 路由

### 自定义 API 路由文件

为了方便管理 API，希望把 API 的路由文件分离到不同的文件，期望 Laravel 自动载入 *routes/api/* 目录下的所有路由文件。

修改 *App\Providers\RouteServiceProvider*，新增 `$apiRoutesPath` 属性并修改 `mapApiRoutes()` 方法如下：

```
    protected $apiRoutesPath = 'api';
    
    protected function mapApiRoutes()
    {
      $routeRegistrar = Route::prefix('api')
        ->middleware('api')
        ->namespace($this->namespace);
        
      load_phps(
        route_path($this->apiRoutesPath),
        function ($file) use ($routeRegistrar) {
          $routeRegistrar->group($file);
        }
      );
    }
```

其中 `load_phps` 是我自定义的全局辅助函数，在 Laravel 启动时已经载入，定义如下：

```
    function load_phps(string $path, \Closure $callable) {
      if (! file_exists($path)) {
        excp("PHP files path not exists: {$path}");
      }
      
      $result = [];
      $fsi = new \FilesystemIterator($path);
      foreach ($fsi as $file) {
        if ($file->isFile()) {
          if ('php' == $file->getExtension()) {
            $result[$file->getPathname()] = $callable($file);
          }
        } elseif ($file->isDir()) {
          $_path = $path.'/'.$file->getBasename();
          load_phps($_path, $callable);
        }
      }
      
      unset($fsi);
      
      return $result;
    }
```

## 模型

### 自定义模型属性

在一个模型控制器获取一个主要的模型数据后，往往还需要和其他接口或模型或计算结果进行聚合，为了方便直接返回一个模型，可以考虑动态地往模型现有数据中添加其他数据。

- 创建一个 `AttrCustomable` trait

```
    <?php
    // Make Model capable to add custom attributes in Laravel 5.5
    namespace App\Traits;
    trait AttrCustomable
    {
        private $customAttributes = [];
        public function addCustomAttribute(string $key, $value)
        {
            $this->customAttributes[$key] = $value;
            return $this;
        }
        public function toArray()
        {
            $data = parent::toArray();
            $data = array_filter(array_merge($data, $this->customAttributes));
            return $data;
        }
    }
```

- 在模型里使用该 trait

```
    <?php
    namespace App\Models;
      
    class User extends Model
    {
      use AttrCustomable;
    }
```

- 在控制器里添加聚合数据

```
    <?php
    
    namespace App\Http\Controllers\Api\V1;
    
    class User
    {
      public function show()
      {
      	return $this->user()
            ->addCustomAttribute('pionts', 10000)
            ->addCustomAttribute('foo', 'bar');
      }
    }
```

### 获取隐藏属性

假设在 User 模型的某个方法内，要访问 `hidden` 过的属性 `password`，有如下几种方式：

- 通过模型的 `makeVisible` 方法 (5.1 是 `withHidden`)

```
    $this->makeVisible('password');    // 使 password 属性可见
    echo $this->password;    // 输出用户密码
    $this->makeHidden('password');    // 使 password 属性隐藏
```

源码入下:

- `makeVisible`

```
    public function makeVisible($attributes)
    {
        $this->hidden = array_diff($this->hidden, (array) $attributes);

        if (! empty($this->visible)) {
            $this->addVisible($attributes);
        }

        return $this;
    }
```

- `makeHidden`

```
    public function makeHidden($attributes)
    {
        $attributes = (array) $attributes;

        $this->visible = array_diff($this->visible, $attributes);

        $this->hidden = array_unique(array_merge($this->hidden, $attributes));

        return $this;
    }
```

- 通过模型的 `getAttributes` 方法

```
    $allAttrs = $this->getAttributes();    // 直接获得模型所有属性（无视可见性设置）
    echo $allAttrs['password'];    // 输出用户密码
```

- 如果模型实现了 `Illuminate\Contracts\Auth\Authenticatable` 接口，则还可以通过该接口提供的 `getAuthPassword` 方法只获取密码属性
    
```
    class User extends Model implements \Illuminate\Contracts\Auth\Authenticatable
    {
      public function index()
      {
        echo $this->getAuthPassword();    // 输出用户密码
      }
    }
```

### 如何判断模型是否存在

```
    $mode->exists;    // !!! 属性调用而非方法
```

### 定义资源路由后控制器模型注入失效？

- 路由定义：

```
    $router->group([
      'prefix' => 'members',
      'middleware' => [
        'api.auth',
      ],
    ], function ($router) {
      $router->resource('address', 'MemberAddress');
    });
```

生成路由定义如下：

```
    - GET|HEAD     /members/address             MemberAddress@index
    - POST         /members/address             MemberAddress@store
    - GET|HEAD     /members/address/{address}   MemberAddress@show
    - PUT|PATCH    /members/address/{address}   MemberAddress@update
    - DELETE       /members/address/{address}   MemberAddress@destroy
```

- 以对应的 `MemberAddress` 控制器的 `show` 方法举例：

请求 `GET` `/members/address/1`，其中 1 号的记录在数据库是实际存在的，可在方法中不是期望的输出：

```
    use App\Models\MemberAddress as Address;
    
    public function show(Address $address)
    {
      dd($address->exists);    // 输出 false !!!
      
      dd(func_get_args());     // 输出如下：
      
      array:2 [▼
        0 => MemberAddress {#365 ▶}
        1 => "1"
      ]
    }
```

结果显示，这里在向控制器方法自动注入模型的时候，参数和模型没有对应上，导致 $address 只是个空模型，这当然不是我们想要的。

在 dingo api GitHub 的这个 [`issue`](https://github.com/dingo/api/issues/1179) 的提示下，去翻了 `bindings` 这个中间件的源码后找到了我想要的解决办法：

> 参见：`Illuminate\Routing\ImplicitRouteBinding@resolveForRoute`。

- 修改 `MemberAddress` 控制器的构造方法如下：

```
    public function __construct()
    {
      $this
        ->middleware('id_filter:address,\App\Models\MemberAddress')
        ->only([
          'show', 'update', 'destroy'
        ]);
    }
```

- 新增中间件并注册到 `App\Http\Kernel`：

```
    protected $routeMiddleware = [
      // ...
      'id_filter' => \App\Http\Middleware\IDFilter::class,
    ];
```

- `IDFilter` 中间件内容如下：

```
    <?php
    // Filter ID for resource routes
    // - 1. Check if has given id key
    // - 2. Check id value illegality
    // - 3. Implicit bind route parameters segment (id key) with given model

    namespace App\Http\Middleware;
    
    class IDFilter
    {
        // TODO
        // !!! Find a better way to handle method params count
        public function handle(
            $request,
            \Closure $next,
            string $idkey,
            string $class = null,
            string $abortOn404 = 'no',
            string $forget = 'no'
        )
        {
            $route = $request->route();
            $id    = ($route->parameters[$idkey] ?? false);
            if (empty_safe($id) || (! ispint($id)) || ($id < 1)) {
                abort(403, "Bad id value of `{$idkey}`: {$id}");
            }
            if ($class) {
                if (! class_exists($class)) {
                    abort(503, 'Model class not exists: '.$class);
                }
                $model  = app($class);
                $_model = $model->where($model->getRouteKeyName(), $id)->first();
                if (ci_equal('yes', $abortOn404) && (! $_model)) {
                    abort(404, 'Model object not found: '.get_class($model));
                }
                if (ci_equal('yes', $forget)) {
                    $route->forgetParameter($idkey);
                } else {
                    $route->setParameter($idkey, $_model ?? $model);
                }
            }
            return $next($request);
        }
    }
```

其中，`getRouteKeyName` 默认返回 `id` 可以在相应模型中重写这个方法实现自定义。

之所以不直接用 Laravel 提供的默认 `bindings` 中间件，原因有两点：

- 没有提供（或者我暂时未找到） route segment 过滤的东西，默认找不到直接返回了 404，而我要的是既能正确提示路由参数格式不对，以及在参数格式正确但找不到数据库记录时，仍然要注入一个空模型到控制器方法。

- 使用了模型的 `resolveRouteBinding()` 方法，该方法在关联不上路由参数时返回 null 导致进入控制器后还需要再实例化一遍。

## 使用锁的场景

通常在并发场景下，**为了防止某个资源被重复创建／更新，在同时存在查询-更新／创建操作的时候**必要使用悲观锁：

```
    $amount = 100;
    \DB::beginTransction();
    $user = \App\Models\User::lockForUpdate()->find(1);    // 锁住该用户对应的行记录，防止并发中的其他请求修改该用户状态
    $log = \App\Models\BalanceLog::insert([
      'before'    => $user->balance,
      'create_at' => time(),
      'amount'    => $amount,
    ]);
    $user->balance += $amount;
    $user->save();
    \DB::commit();
```

如果不使用 `lockForUpdate()` 来锁住该用户对应的记录，那么在多个相同请求到来时，该用户的余额和日志会被更新／创建多条，这明显不是我们想要的。

`lockForUpdate()` 创建的锁将在本次事务结束后释放，且如果记录不存在，`lockForUpdate()` 则不会起到什么作用。



# <u>[Laravel-Admin](https://github.com/z-song/laravel-admin)</u> 

> 不需要前端、UI, 后端就可以开发整个后台

这是个专门写后台管理系统的轮子。简而言之，是用 PHP 同时写后台页面和后端逻辑。

优点是将 Web 前端组件很好地封装到后端，使后端人员可以不用操心页面上的东西，要什么页面元素按照文档照做就行了，这么一来，写一个后台只需要后端就够了。

# <u>[Dingo API](https://github.com/dingo/api/)</u>

虽然对于 Laravel 5.5 来说，要实现 Dingo API 的所有功能易如反掌，但之所以选择 Dingo API，主要是因为已经有很多的人用它，相信它封装得更标准，更简便，更稳定，不想花时间重复造轮子罢了。

如果使用过程发现并不好用，或者不太适合我们的项目，这样改造起来也有针对性。

## 优缺点

- 优点：可以自动生成文档

Dingo API 使用提供了 api:docs 这个 artisan 命令，可以自动为 API 生成版本文档。

- 缺点：文档生成不够智能，代码中注释量增多

虽然在定义 API 端点的时候指已经明了版本号和路由，但是为了生成更好看的 API 文档，仍然需要在类和方法前面重新定义路由的类型和路径，已经版本号。这样一来，代码中要添加很多额外的注释。

- 缺点：`api:cache` 会缓存除了 API 以外的所有路由。

期望:只缓存通过 Dingo API 定义好的路由。

## FAQ

- `version` 为 `group` 的别名

```
    // 获得版本号为 v1 的路由列表
    dd(version('v1'));
```

### 如何访问指定版本的接口？

在配置 Dingo API 的时候，有个配置项叫做 `API_VERSION` 的配置，这个配置的作用是作为默认版本号，即客户端当没有指定。

在 HTTP 请求头中添加 `Accept` 字段，举例说明：

```
    curl -X GET http://api.example.com/user \
    -H 'Authorization: Bearer ??TOKEN??' \
    -H 'Accept: application/vnd.{API_SUBTYPE}.{VERSION_NUMBER}+json' \
```

其中，`API_SUBTYPE` 就是 `.env` 中配置的 `API_SUBTYPE` ，`VERSION_NUMBER` 就是使用 `version()` 方法定义过的版本号，`vnd` 是 `API_STANDARDS_TREE` 推荐的值，`json` 代表客户端期望服务器返回的数据格式。

### 如何正确使用自动刷新 Token 机制？

需要前后端协作好：后端要根据当此请求携带的 Token 自动生成一条新 Token，并返回给前端；前端要做好自动使用后端每次返回的更新 Token 来作为下次 API 请求的 Token。

以 <u>[tymon/jwt-auth](https://github.com/tymondesigns/jwt-auth)</u> 认证驱动为例，在 `Tymon\JWTAuth\Providers\AbstractServiceProvider` 中已经定义好 `$middlewareAliases` 中间件别名如下：

```
    protected $middlewareAliases = [
      'jwt.auth' => Authenticate::class,
      'jwt.check' => Check::class,
      'jwt.refresh' => RefreshToken::class,    // 重新刷新 TOKEN
      'jwt.renew' => AuthenticateAndRenew::class,
    ];
```

可以直接在路由定义或控制器构造函数中使用：

```
    // 路由中使用
    dingo()->version('v1', [
        'namespace'  => 'App\Http\Controllers\Api\V1',
        'middleware' => [
            'jwt.refresh',
        ],
    ], function ($router) {
      // ...
    });
    
    // 在控制器中使用
    public function __construct()
    {
      $this->middleware('jwt.refresh');
    }
```

每次请求 API 成功后，均会在响应头中的 `Authorization `字段，其值便是刷新后的 `Token`。

### 如何使用 dingo Router 中 `name()` 定义过的 API 路由？ 

Dingo 定义过的的命名路由是不能用 Laravel 自带的 route 方法获取完整 URL 的，只能用 Dingo 的 URL 生成器。

**举例说明：**

```
    // 定义微信授权登录回调路由
    $router
      ->post('{user}/wechat', 'Passport@oauthloginFromWechat')
      ->name('callback.oauth.wechat');
      
    // 错误写法
    route('callback.oauth.wechat');    // 提示找不到路由
    
    // 正确写法
    app('Dingo\Api\Routing\UrlGenerator')
      ->version('v1')
      ->route('callback.oauth.wechat', ['user' => 1]);
```

# 参考

- <u>[liyu001989/lumen-api-demo](https://github.com/liyu001989/lumen-api-demo)</u>

- <u>[Laravel 多用户认证](https://b.von.im/2017/07/15/2017-7-15-Laravel-Multi-Auth/)</u>

- <u>[Where to write transformers ? · Issue #341 · dingo/api](https://github.com/dingo/api/issues/341)</u>

- <u>[Test Driven API Development using Laravel, Dingo and JWT with Documentation](https://m.dotdev.co/test-driven-api-development-using-laravel-dingo-and-jwt-with-documentation-ae4014260148)</u>