---
title: Lumen5.4-开发记录-转
date: 2018-01-05 15:16:14
categories: Lumen
tags: [Lumen, API, Framework]
---

### 惠吃猫-重构-API接口

> 基于二次开发,无法继续开发后的一次重构,使用lumen重构API接口


<!--more-->

### 引入多个 route 文件

在 routes 文件夹下创建一个路由文件，比如 api.php，然后在 bootstrap/app.php 中找到并添加：

```php
    $app->group(['namespace' => 'App\Http\Controllers'], function ($app) {
        require __DIR__.'/../routes/web.php';
        require __DIR__.'/../routes/api.php';    // 新增的路由文件
    });
```

### 为路由定义可选参数

Lumen 不支持。

### 在中间件中获取路由参数

**如下两种 Lumen 不支持但 Laravel 支持：**

```
    public function handle($request, Closure $next)
    {
        $request->route()->parameters();
        $request->route('parameter');
    }
```

`Lumen` 只能使用：`$request->route()[2]` 这个数组，里面就是路由参数。

### 路由定义和命名空间
```
    <?php
    $app->group([
        'namespace' => 'ThirdParty',
    ], function () use ($app) {
        $app->group([
            'prefix'     => 'qiniu',
            'middleware' => [
                'qiniu_auth',
            ]
        ], function () use ($app) {
            $app->group([
                'middleware' => [
                    'jwt_auth',
                ],
            ], function () use ($app) {
                $app->get('up_ticket', 'Qiniu@getUploadTicket');
            });
            $app->post('upload_cbk', [
                    'as'   => 'qiniu_upload_callback',
                    'uses' => 'Qiniu@uploadCallback',
                ]);
        });
    });
```

> 在这种路由定义方式中，因为路由文件中默认的命名空间是 `App\Http\Controllers`，因此如果需要给路由定义 `namespace`，比如这里的 `ThirdParty`，则无需再写成：`App\Http\Controllers\ThirdParty`，因为它会自动在 `App\Http\Controllers` 命名空间下找 `ThirdParty` 这个路径，不会报错。

> 同理，上面的 `Qiniu@getUploadTicket` 也无需写成 `App\Http\Controllers\ThirdParty\Qiniu@getUploadTicket`，因为在这个群路由定义的开始已经指定为 `ThirdParty` 了，下级在找路由器的时候便会自动去 `App\Http\Controllers\ThirdParty` 这个命名空间找，这样一层一层，以此类推。


### 重定向至命名路由并传递参数

```
    return redirect()->route('admin', ['key' => 'value']);
```

### 启用自定义 JWT 认证

- 首先在 `bootstrap/app.php` 

```
    $app->routeMiddleware([
        'auth_jwt' => App\Http\Middleware\Auth\JWT::class,
    ]);
```

- 然后修改 `app/Http/Middleware/Auth/JWT.php` 为：

```php
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

### Pass value from middleware to controller
> 将中间件的值传递给控制器

```
    // 1. Middleware
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

### 自定义配置文件

- `bootstrap/app.php`

```
    $app->configure('custom');
```

- 创建一个 `config/custom.php` 文件

```
    <?php
    return [
    	'key' => 'value'
    ];
```

- 使用

```
    dd(config('custom'));
    /* output: 
    array:1 [▼
      "key" => "value"
    ]
    */
```

### `CORS`

如果是客户端是浏览器，则在调用 `API` 的时候，基本上无法避免跨域问题。

不过处理也很简单，需要浏览器和服务器同时配合：浏览器在发起 OPTIONS preflight 请求的时候，服务器返回认可的一些信息就行了。

具体如下：

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
 
- `jQuery AJAX Demo`

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
 
### `migration` 中给表加注释

```
    public function up()
    {
      Schema::create('table_name', function (Blueprint $table) {
        // DDL (数据定义语言)
      });
      DB::statement('ALTER TABLE `table_name` comment "table_comment"');
    }
```

### `migration` 设置 `current_timestamp`

```
    $table->dateTime('create_at')->default(
        DB::raw('CURRENT_TIMESTAMP')
    );
```

### 连接多个数据库

- 初始化数据库配置对象
 
```
    # bootstrap/app.php
    $app->configure('database');
```
 
- 创建并修改 `config/database.php`

```
    <?php
    return [
    	// ...
      
        'default'     => 'mysql_db1_rw',
        'connections' => [
          
          	// ...
          
            'mysql_db1_rw'  => [
                'driver'    => 'mysql',
                'host'      => env('DB_HOST'),
                'database'  => 'db1_rw',    // read and write
                'username'  => env('DB_USERNAME'),
                'password'  => env('DB_PASSWORD'),
                'charset'   => 'utf8',
                'collation' => 'utf8_unicode_ci',
            ],
            'mysql_db1_ro'  => [
                'driver'    => 'mysql',
                'host'      => env('DB_HOST'),
                'database'  => 'db1_ro',    // read only
                'username'  => env('DB_USERNAME'),
                'password'  => env('DB_PASSWORD'),
                'charset'   => 'utf8',
                'collation' => 'utf8_unicode_ci',
            ],
          
            // ...
        ]
      
      	// ...
    ];
```

**注意：**

一旦创建了 `config/database.php` 文件，系统默认的 `vendor/laravel/lumen-framework/config/database.php` 将失效。

因此，最好的做法是拷贝系统自带的配置文件到项目根目录，然后进行一些自定义的修改。

```
    rm -rf config/*.php
    cp vendor/laravel/lumen-framework/config/database.php config/
```

- 在控制器中使用不同的数据库连接

根据上面的配置默认会连接 `mysql_db1_rw` 这个数据库。那如果需要连接第二个数据库，使用示例如下：

```
    // Use default connection => mysql_db1
    app('db')->connection()->select('show tables');
    DB::connection()->select('show tables');
    // Use mysql_db2 connection
    app('db')->connection('mysql_db1_ro')->select('show tables');
    DB::connection('mysql_db1_ro')->select('show tables');
```

### 自定义日志文件名

> `Lumen` 中默认的文件名是被硬编码在 `Application` 类中了的，要实现这样的效果，必须重写 `Laravel\Lumen\Application@getMonologHandler()` 方法。

- 新增文件：`app/Application.php`

```php
    <?php
    
    // Custom default or hard-coded behavior of lumen
    // hard-coded behavior 硬编码
    
    namespace App;
    
    use Laravel\Lumen\Application as LumenBase;
    use Monolog\Formatter\LineFormatter;
    use Monolog\Handler\StreamHandler;
    use Monolog\Logger;
    
    class Application extends LumenBase
    {
        // Rewrite log handler
        protected function getMonologHandler()
        {
            return (
                new StreamHandler(storage_path(env(
                    'APP_LOG_PATH',
                    'logs/'.date('Y-m-d').'.log'
                )),
                Logger::DEBUG
            ))
            ->setFormatter(new LineFormatter(null, null, true, true));
        }
    }
```

- 修改 bootstrap/app.php
```
    $app = new App\Application(
        realpath(__DIR__.'/../')
    );
```

### 自定义 artisan 命令

- 创建一个 command 类

```
    <?php
    // app/Console/Commands/Hello.php
    namespace App\Console\Commands;
    use Illuminate\Console\Command;
    class Hello extends Command
    {
        protected $name = 'patch:hello';
        protected $description = 'Balabalabala';
        
        public function fire()
        {
            $this->info('hello atisan cmd');
        }
    }
```
 
- 创建一个 command service provider

```
    <?php
    // app/Providers/CommandService.php
      
    namespace App\Providers;
    use Illuminate\Support\ServiceProvider;
    class CommandService extends ServiceProvider
    {
        public function register()
        {
            $this->app->singleton('command.patch.hello', function () {
                return new \App\Console\Commands\Hello;
            });
            $this->commands(
                'command.patch.hello'
            );
        }
    }
```
 
- 注册该 `command service provider`

```
    // bootstrap/app.php
    $app->register(App\Providers\CommandService::class);
```
 
- 测试

```
    php artisan patch:hello
    // output: hello atisan cmd
```
 
### 模型查询结果中隐藏部分字段

```
    protected $hidden = [
      'passwd',
    ];
    protected $visible = [
      'uid'
    ];
```

其中 $visible 的优先级比 $hidden 高。


### FAQ

- docker 搭建的运行环境，macOS 下执行 php artisan migrate 提示：_php_networkgetaddresses: getaddrinfo failed: nodename nor servname provided, or not known？
 
 这是因为是在本地（macOS）执行的这条命令，而本地对 .env 中配置的 DB_HOST=db 是解析不了的。因此会出现这个错误。
 
 解决这种问题也很简单，有两种办法：
 
- 修改主机 hosts 文件，使 db 指向 mysql 容器的 域名／IP。比如：127.0.0.1 db。（推荐）

- 进入 docker 容器再运行迁移。
  
- Class ‘Memcache’ not found?
 
安装 memcached/memcache 扩展。
 
- php artisan migrate PDOException: user not exist?
   - 检查是主机地址是否填错。
   - 检查配置中的数据库服务器是否存在该用户。
   
### 可能踩的坑
 - 数据库事务操作时部分逻辑未生效？
 
 检查对象是否是有效的 Model 对象。举例说明，以下代码第一步将不会生效：
 
 ```
    $user = User::select('money')->whereUid(1)->first();
    \DB::beginTransaction();
    // 第一步：减少用户余额
    $user->money -= 100;
    $decreased = $user->save();
    // 第二步：记录日志
    $loggedId = Log::insertGetId([
      'uid'      => 1,
      'amount'   => -10,
      'intro'    => 'test',
    ]);
    // 两步结果共同决定是否执行事务
    if ($decreased && $loggedId) {
      \DB::commit();
    } else {
      \DB::rollback();
    }
 ```
 
 这是因为，对模型对象调用了 select() 方法后，就不是一个完整的模型对象了（从结果来看）。
 
 解决办法是，不使用 select() 或者使用 find()。
 ```
    $user = User::whereUid(1)->first();
    // Or
    $user = User::find(1);
 ```
 
### 参考
 - [Laravel Middleware return variable to controller](https://stackoverflow.com/questions/30212390/laravel-middleware-return-variable-to-controller)
 - [Documentation - Lumen 5.4](https://lumen.laravel.com/docs/5.4)
 - [Documentation - Laravel 5.5](https://laravel.com/docs/5.5)
 - [Laravel_API-Manual](http://blog.cjli.info/2017/07/09/Lumen5.4-Exp/#more)
 - [Laravel 5 Cheat Sheet](https://learninglaravel.net/cheatsheet/)
 - [Laravel 5.1 LTS速查表](https://cs.laravel-china.org/)
 - [Configuring Multiple Database Connections In Lumen](https://paulund.co.uk/configuring-multiple-database-connections-laravel)
 - [Enable Custom Configuration Files in Lumen](https://blog.ryanrampersad.com/2015/05/26/custom-config-lumen/)
 - [Lumen Recipes :: Creating a New Artisan Command](http://codrspace.com/MattWohler/creating-a-new-artisan-command-with-lumen/)
 - [DB:select toArray()](https://laravel.io/forum/10-14-2014-dbselect-toarray)