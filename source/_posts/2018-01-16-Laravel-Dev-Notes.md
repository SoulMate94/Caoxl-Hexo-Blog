---
title: Laravel 使用笔记
date: 2018-01-16 15:01:17
categories: Laravel
tags: [PHP, Laravel, Framework]
---

# Laravel Dev Notes

> 学习别人的博客,并且转载的...


# 路由

## as 在路由定义中的作用？如何使用？如何向命名过的含参数的路由传递参数？

> 命名路由让你可以更方便的「为特定路由生成 URL 」或「进行重定向」。


<!--more-->


```
    # 1. 定义路由 并为其命名
    # 第一种 => 通过数组键 `as` 命名
    Route::get('user/profile/{id}', ['as' => 'profile', function () {
        echo 'user/profile/{id}';
    }]);
    # 命名路由群组和子路由
    Route::group(['as' => 'admin::'], function () {
        Route::get('dashboard', ['as' => 'dashboard', function () {
            // 路由名称为「admin::dashboard」
        }]);
    });
    
    # 第二种 => 通过链式调用 `name()` 方法命名
    Route::get('user/profile/{id}', 'UserController@showProfile')->name('profile');
    # 2. 使用命名过的路由
    Route::get('test_profile', function () {
      return redirect()->route('profile', ['id' => 1]);    // 这里的 `profile` 就是某条路由定义过的命名; `id` 就是该路由中的参数名
      
      return redirect()->route('admin::dashboard');    // 重定向到路由群组命名过的子路由
      # Or
      # return redirect()->to(route('profile'));
    });
```

## 路由参数限制

 - 可选路由参数
 ```
    # 参数名后面 结束 `}` 之前的 `?` 代表此路由参数为可选参数
    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });
 ```
 
 - 正则过滤路由参数：未通过正则检查的路由参数将会出现 NotFoundHttpException 报错
 ```
    # 局部限制
    Route::get('user/{id}', function ($id) {
        // ...
    })->where('id', '[0-9]+');
    # 全局限制 => boot()@/app/Providers/RouteServiceProvider.php
    public function boot(Router $router)
    {
        $router->pattern('id', '[0-9]+');    // 模式一旦被定义，便会自动应用到所有使用该「参数名称」的路由上
        parent::boot($router);
    }
 ```
 
## url() 和 route() 的区别

 `url()` 可以生成任意的 URL，不检查是否是合法路由。
 
 `route()` 则会接受命名过的路由别名，然后生成 URL。如果路由别名不存在，则会报错。
 
## 关于子域名路由

> 子域名可以像路由 URIs 分配路由参数。

```
    Route::group(['domain' => '{account}.myapp.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });
```

这里需要额外说明的是，要使用上子域名这个功能，必须要在域名解析那里将所有具体的子域名指向和主域名同样的服务器（泛解析），同样的网站路径下，才有意义。

 - 路由群组共用参数和嵌套群路由
 
 在路由的第一个数组数组参数中的 prefix 键，代表的是「当前群路由」中共用的路由前缀，其值也可以包含参数：
 
 ```
    Route::group([
    	'as'     => 'admin::',
    	'prefix' => 'admin',
    	], function () {
        Route::get('dashboard', ['as' => 'dashboard', function () {
            // 路由名称为「admin::dashboard」
            // URL => route('admin::dashboard');
        }]);
        // 嵌套群路由
        Route::group([
        	'prefix' => 'accounts/{account_id}'     // 路由群组共用参数
        	], function () {
    	    Route::get('detail', function ($account_id)    {
    	        // URL => /admin/accounts/{account_id}/detail
    	    });
    	});
    });
 ```
 
# 中间件

## POST API 中禁用 CSRF 中间件

```
    // App\Http\Middleware\VerifyCsrfToken
    protected $except = [
      '/api/*'
    ];
```

## 自定义 Auth 中间件

 - 创建类文件
 ```
    php artisan make:middleware UserAuth
    php artisan make:middleware AdminAuth
 ```
 
 - 编写 handle() 方法
 ```
    // AdminAuth
    public function handle($request, Closure $next)
    {
      if (!session()->has('admin')) {
        return redirect()->to('/admin/login');
      }
      return $next($request);
    }
    // UserAuth
    public function handle($request, Closure $next)
    {
      if (!session()->has('user')) {
        return redirect()->to('/login');
      }
      return $next($request);
    }
 ```
 
 - 注册／挂载到应用内核 $routeMiddleware
 ```
    // app\Http\Kernel.php
    portected $routeMiddleware = [
      // ...
      'auth.admin' => \App\Http\Middleware\AdminAuth::class,
      'auth.user' => \App\Http\Middleware\UserAuth::class,
    ];
 ```
 
 - 路由中调用新中间件
 ```
    Route::group([
      'prefix'     => '/profile',
      'middleware' => 'auth.user',
    ], function () {
      // ...
    }
    Route::group([
      'prefix'     => '/admin',
      'middleware' => 'auth.admin',
    ], function () {
      // ...
    }
 ```
 
# 数据库

## 查询构造器
## distinct select
必须 select 中指定之后 distinct 才有效。

## 模型也能使用其方法
在 Laravel 模型中，也能使用和查询构造器一样的一些方法。

## 往数据库插入测试数据
 - 通过 seeder => 直接修改 database/DatabaseSeeder.php
 ```
    public function run()
    {
      Model::unguard();
      DB::table('users')->insert([
        [
          'name'     => str_random(10),
          'email'    => str_random(10).'@gmail.com',
          'password' => bcrypt('secret'),
        ],
        // more ...
      ]);
      Model::reguard();
    }
 ```
 
 - 通过调用 seeder:
   - 生成独立的 seeder：php artisan make:seeder UsersTableSeeder
   - 修改 UsersTableSeeder 中要填充的内容和数量
   
 ```
        public function run()
        {
          DB::table('users')->insert([
            [
              'name'     => str_random(10),
              'email'    => str_random(10).'@gmail.com',
              'password' => bcrypt('secret'),
            ],
            // more ...
          ]);
        }     
 ```
 
 - DatabaseSeeder 中调用: call(UsersTableSeeder);
 
 ```
    public function run()
    {
      Model::unguard();
      $this->call(UserTableSeeder::class);
      Model::reguard();
    }
 ```
 
 - 通过模型工厂：
   - 将要填充数据的表对应的模型及其格式注册到 databse/factories/ModelFactory.php
   
 ```
    $factory->define(App\Models\User::class, function (Faker\Generator $faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->safeEmail,
            'password' => bcrypt(str_random(10)),
            'remember_token' => str_random(10),
        ];
    });
 ```
 
  - 然后在 DatabaseSeeder 中调用 factory()->times()->make()
  - 最后通过模型类一次性插入生成的测试数据
  
 ```
    public function run()
    {
      Model::unguard();
      # 默认的方式 但不推荐使用 `create()`
      // factory(App\Models\User::class, 50)->create()->each(function($u) {
      //     $u->posts()->save(factory(App\Post::class)->make());
      // });
      
      # 推荐使用 `make()`
      $users = factory(\App\Models\User::class)->times(100)->make();
      \App\Models\User::insert($users->toArray());
      Model::reguard();
    }
 ```
 
## Migration

## `--create=TABLE_NAME` 和 `--table=TABLE_NAME` 

有什么区别？创建 migrations 时是否需要带上此参数？

带 `--create=TABLE_NAME` 表示在运行 `php artisan migrate` 的时候会创建物理表（表名为 TABLE_NAME）到数据库，不带则只是创建该迁移文件。

带 `--table=TABLE_NAME` 表示在进行修改表操作的时候指定要修改的表名为 TABLE_NAME。

最后，对应到 migrations 文件中的 up() 方法，`--create=TABLE_NAME` 调用的是 `Schema::create('TABLE_NAME')`，而 `--table=TABLE_NAME` 则调用的是 `Schema::table('TABLE_NAME')`。

*说明:*
如果要运行由不带 `--create=TABLE_NAME` 的命令创建的 migrations，只需要修改这些文件的 up 方法（看情况，已有内容的可以不用在修改，但是通常都需要修改），即使用 Schema 语法写入需要的数据库操作，最后重新执行以下操作即可：

```
    # 1. 修改数据库 migrations 表中要运行的 migration 的 batch 值为当前最高值+1
    -- mysql command statart
    update `migrations`
    set `batch` = (batch + 1)
    where `migration` = '2014_10_12_000000_create_users_table';
    -- mysql command end
    # 2. 回滚已手动置新的 migration 最新批次
    php artisan rollback
    # 3. 重新执行迁移
    php artisan migrate
```

这里的 batch 代表的是执行 php artisan migrate 的批次，即是记录 laravel 是第几批运行某个具体的迁移的。

batch 值相同的 migrations 在执行 php artisan migrate:rollback 时都会被回滚。

## 关于 `up()` 和 `down()`

这两个方法必须要是互逆的。

即使说，up() 里面写的是 `create`，`down()` 里面就必须写 `drop`，而不能写 `renameColumn()` 之类的，反之同理。

## Eloquent

- 关于调用含有 hasMany() 和 belongsTo() 关系的方法

实际调用的时候不能当作方法调用，而要当作属性获取。

即不能写成类似 `$user->tasks()` 而要写成 `$user->tasks`。

此外，含有这些关联属性的方法名的单复数形式要和关系相对应，即如果是 `belongs-to` 关系，则模型中相应的方法名应该是`单数`，如果是 `has-many` 关系，则模型中相应的方法名应该是复数，举例说明：

```
    # App\Models\Task.php
    public function user() {
      return $this->belongsTo(User::class);
    }
    # App\Models\User.php
    public function tasks() {
      return $this->hasMany(Task::class);
    }
```

- 什么时候适合用 Eloquent？

Eloquent 在对常见数据库操作时是非常得心应手的，比如单表的增删改，表的查询（包括联表），都是 OK 的，首选 ORM 比较合适。

但是，在一些增删改完整性要求高的情况下，比如事务，这时候 Eloquent 是没办法做到的，这时候只能使用 DB Facade 提供的事务处理方法。

- `$fillable`／`$guarded` and Mass Assignment？

Mass Assignment，或称批量复制，指的是在 Laravel 在创建 Model 时可以批量传入一个数组，一次性设置好要保存的属性，而不用一个个指定字段具体的值，比如可以直接快速创建一个用户： $user = new User(Input::all());。

`$fillable` 和 `$guarded` 都是在批量赋值过程中使用的， `$fillable` 相当于的白名单，`$guarded` 相当于黑名单，使用时只能使用其一。 **白名单总能覆盖黑名单**。

默认情况下，模型的所有属性都是黑名单，只有 $fillable 中的属性才能在批量赋值时被插入数据库。

但是，并不是说 $fillable 之外的属性都不能被赋值，只是，不能批量赋值，可以单独为一些特殊字段赋值，然后保存：

```
    $user->user_type = 'admin';
    $user->save();
```

当要批量赋值 $fillable 中没有指定的属性时，可以使用 forceCreate() 硬性创建，只是需要确保特殊字段的安全性。

## 团队数据库开发的工作流简要说明

此流程相关的元素有：migration & seeder|factories & git，下面举例说明：

- A 在其本机建新表 users，插入了一些测试数据，最后生成了 migrations，seeds 或者 factories 文件
```
    # 1. 创建迁移
    php artisan make:migration create_users_table --create=users
    # 2. 根据需要修改 migrations 中的定义
    # 3. 运行迁移 => 插入该迁移中定义的表到数据库
    php artisan migrate
    # 4. 为该表插入测试数据（具体操作详见：@往数据库插入测试数据）
    # 5. 提交到 git 仓库
    git add -A && git commit -m "create table users"
    git pull origin dev # 根据实际情况看是否需要合并冲突
    git push origin own_branch
```

- B 拉取到表 users 的结构和 seeder，然后同步新改动到本机
```
    # 1. 拉取仓库最新改动
    git pull origin dev    # 假设协作分支为 dev（根据情况看是否需要先提交本地改动）
    # 2. 迁移数据库更新（同步）
    php artisan migrate
    # 3. 同步测试数据（可选）
    php artisan db:seed
```

- B 为 users 表新增一个 gender 字段，也插入了一些测试数据，最后生成了 migrations, seeds 或者 factories 文件
```
    # 1. 创建迁移
    php artisan make:migration add_gender_to_users_table --table=users
    # 2. 修改迁移定义 为表 users 表新增 gender 字段
    # 3. 运行迁移（同上）
    # 4. 为该表插入测试数据（同上）
    # 5. 提交到 git 仓库（同上）
```

最简单的工作流程就是这样，如果有更多的开发者，也是按照这个基本流程来同步数据库改动。

可以看出，通过 laravel，多人同步数据库版本的核心其实就只有 php artisan migrate 这一条命令，包括最后部署到服务器，也只需要 migrate 一下就可以了，个人认为是很简单。

*说明:*
如果对数据库有新的改动，则必须要创建新的迁移。最好不要修改已经迁移过的 migrations，因为如果要同步这些已修改的迁移，则需要先将这个迁移手动置最新一批，然后回滚，再执行 php artisan migrate 才能生效。

但是，由于保存 migrations files 的迁移批次是存在数据库中的，而 A 的数据库表 migrations 并不会同步到 B 数据库的 migrations 表，因此如果 B 得到了 A 修改过已迁移过的 migrations files 也需要先将这个迁移手动置最新一批，然后回滚，再执行 `php artisan migrate` 才能生效

所以，**数据库每发生一次改动，最好是新建一次迁移**。

# 认证

## 启用 auth 认证的前提

## Users 表必须字段
 - 可以唯一标志一个用户的账号：至少有一个，可以是 username，也可以是 email，手机号码等。
 - password：> 60 个字符。
 - remember_token：nullable 、100 个字符。（migration class: $table->rememberToken()）

## 表单
 - <input name="email" type="text">
 - <input name="password" type="password">
 - <input name="remember" type="checkbox">
 
表单需要的 name 值具体是什么，取决于在 PHP 中所定义的接受 Key。比如，可以把 email 换成 account，并使用户无论输入邮箱还是用户名都可以登录成功。

```
    /**
      * 获得登录凭证
      * @overide
      */
    protected function getCredentials(Request $request)
    {
      $login = $request->get('account');
      $field = filter_var($login, FILTER_VALIDATE_EMAIL) ? 'email' : 'name';
      return [
        $field => $login,
        'password' => $request->get('password'),
      ];
    }
```

## 配置

- 默认规定的路由格式，重定向位置，认证相关视图路径（默认在 auth/ 下面）都是可以自定义的。

```
    # routes.php
    Route::post('login', 'Auth\AuthController@postLogin');
    # AuthController Class
    protected $username = 'account';      // 用户账户的唯一标志
    protected $redirectPath = '/home';    // 用户登录成功后的重定向路由
    # Authenticate Class => handle()
    public function handle($request, Closure $next)
    {
      if ($this->auth->guest()) {
        if ($request->ajax()) {
          return response('Unauthorized.', 401);
        } else {
          return redirect()->guest('auth/login');     // 表单形式登录校验失败后重定向路由
        }
      }
      return $next($request);
    }
```

## 调用认证中间件 Auth 的几个地方

- 路由闭包
```
    # 普通路由
    Route::get('profile', [
      'middleware' => 'auth',
      function() {
        // ...     // 只有认证过的用户能进来这里
    }]);
    # 群路由
    Route::group([
      'middleware' => 'auth'
    ], function () {
      // do something
    });
```

- 控制器构造器
```
    public function __construct() {
      $this->middleware('auth');
    }
```

## 自定义最大登录尝试次数、冻结登录时间和错误提示消息

```
    #  App\Http\Controllers\Auth\AuthController.php
    protected $maxLoginAttempts = 10; // Amount of bad attempts user can make
    protected $lockoutTime = 300; // Time for which user is going to be blocked in seconds
    # resources/lang/en/auth.php
    'failed' => 'These credentials do not match our records.',
    'throttle' => 'Too many login attempts. Please try again in :seconds seconds.',    
```


# 授权

授权的目的，是为了检查某个角色是否具备某个权限。

具体到代码层面，通常是去判断模型 A 是否与模型 B 有所属关系。

> 数据库表的关联字段是否相同，或者专门的权限表中是否有模型 A 和 B 的关联记录，等等。

Laravel （>5.1.11）默认授权工作流程如下：

## 定义权限

通过 Illuminate\Auth\Access\Gate 类，在 AuthServiceProvider 文件中定义应用中的所有权限

```
    namespace App\Providers;
    use Illuminate\Contracts\Auth\Access\Gate as GateContract;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    class AuthServiceProvider extends ServiceProvider
    {
      // 注册应用程序的认证或授权服务
      public function boot(GateContract $gate)
        {
            $this->registerPolicies($gate);
        	// 1. 通过回调处理授权
            $gate->define('edit-post', function ($user, $post) {
                return $user->id === $post->user_id;
            });
        	// 2. 通过类和方法名
        	// $gate->define('edit-post', 'ClassName@methodName');
        }
    }
```

这里的闭包里不用手动检查 $user 是否存在，未登录用户或是没有用 forUser() 方法指定的用户，Gate 自动返回的权限值为 false。

## 定义拦截授权事件

在判断所有已定义权限之前，可以为某些特殊用户，比如最高权限的超级管理员（或者最低权限的访客），中断其后的授权校验，直接判定为拥有最高权限。

```
    $gate->before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });
```

也可以使用 after() 方法定义一个在所有授权检查结束后会被运行的回调，但是无法修改 after 回调中授权检查的结果。

```
    $gate->after(function ($user, $ability, $result, $arguments) {
        //
    });
```

## 检查权限

各种权限定义好后，接下来就是进行权限检查。Laravel 有以下几种检查方式：

## 通过 Gate Facade

Gate facade 包含 3 个方法：check、allows、denies。

以判断当前用户是否有刚刚创建的 edit-post 权限举例说明：

### 检查已登录用户权限

```
    namespace App\Http\Controllers;
    use Gate;
    use App\Modles\User;
    use App\Modles\Post;
    use App\Http\Controllers\Controller;
    class PostController extends Controller
    {
        // 编辑指定的文章
        public function edit($id)
        {
            $post = Post::findOrFail($id);
          	// 1.使用 denies
            if (Gate::denies('edit-post', $post)) {
                abort(403);
            } else {
              // 更新文章...
            }
          
          	// 2.使用 allows 或 check (和 denies 相反)
          	// cheak 是 allows 的别名
            if (Gate::allows('edit-post', $post)) {
              // 更新文章...
            } else {
                abort(403);
            }
        }
    }
```

这里不需要传递当前登录用户至该方法内，因为 Gate 会自动加载当前登录用户。

### 检查指定用户的权限

如果想要检查已登录用户之外的其他用户是否具有指定的权限，可以使用 forUser() 方法：
    
```
    if (Gate::forUser($user)->allows('edit-post', $post)) {
        // do right thing
    }
```

### 多参数情况
    
```
    // 1.权限定义时回调的参数
    Gate::define('delete-comment', function ($user, $post, $comment) {
        // ...
    });
    // 2.同时校验多个权限
    if (Gate::allows('delete-comment', [$post, $comment])) {
        //
    }
```

## 通过 User Model

默认情况下，Laravel 的模型使用了 Authorizable trait，它提供了两个方法：can 及 cannot，类似于 Gate 的 allows和 denies。

还是用上面的例子说明：

```
    // 编辑指定的文章
    public function edit(Request $request, $id)
    {
      $post = Post::findOrFail($id);
      if ($request->user()->cannot('edit-post', $post)) {
        abort(403);
      } else {
    	// 更新文章...
      }
      
      // 或者使用 can 反回来
      if ($request->user()->can('edit-post', $post)) {
    	// 更新文章...
      } else {
        abort(403);
      }
    }
```

## 通过 Blade 模版标签

在 Blade 模板中，你还可以使用 @can 标签来快速检查当前登录用户是否有指定的权限。例子同上：

```
    <a href="/post/{{ $post->id }}">查看文章</a>
    @can('edit-post', $post)
    <!-- 当前登录用户可以编辑文章 -->
    @else
    <!-- 当前登录用户不可以编辑文章 -->
    @endcan
```

## 通过表单请求授权

表单请求 是一个自定义的请求类，里面包含着验证逻辑，需要先创建好表单请求类。

```
    php artisan make:request UpdatePostRequest
```

可以在 UpdatePostRequest 类中的的 authorize() 方法中调用 Gate 进行授权验证。

```
    public function authorize()
    {
        $postId = $this->route('post');
        return Gate::allows('edit-post', Post::findOrFail($postId));
    }
```

自定义请求类编写好后，就可以作为 Laravel 默认的 Request 类传入控制器使用了。

如果 authorize 方法返回 false，则会自动返回一个 HTTP 响应，其中包含 403 状态码，而你的控制器方法也将不会被运行。（这样也简化了控制器代码量）


# 视图

## Blade 标签的几种形态

```
    # 1. 普通形态「实体转义过」
    {{ $var }}
    # 2. 注释形态
    {{-- This is a comment, which will not be in the rendered HTML --}}
    # 3. 原样输出形态
    @{{ whatever you write here will be output in HTML, rather than executed }}
    # 4. 原样执行形态「HTML 实体将不被转义」
    {!! '<script>alert("will be executed")</script>' !!}
```


# 队列

## Laravel 使用队列步骤

- 「创建」任务类
```
    php artisan make:job SendNoticeEmail --queued
```

创建后，根据业务需求编写任务类，通常是某个模块所需的耗时功能。比如：发邮件、截图、生成压缩包、七牛云上传，等。

`--queued` 参数表示运行具有队列属性，即该任务类必须实现 ShouldQueue 接口。

- 「派发」任务

在某个业务逻辑处派发具体的任务，即「任务入队」：`$this->dispatch($job)`。

也支持使用 `$this->dispatchFrom()` 从请求中派发任务。

**只要使用了 DispatchesJobs trait 的类都可以直接通过 $this->dispatch($job) 来派发任务到队列上。**

在将任务入队的时候，可以用 onQueue(QUEUE_NAME) 指定任务所属的队列；也可以用 delay(secs) 来延迟多少秒后运行。

- 处理「回调」

所注册的队列任务，运行完成后会被执行的回调操作，比如：数据库状态更新、邮件通知，等。主要有两种：

完成通过：`Queue::after($connection, $job, $data)`。

失败通过：`Queue::failing($connection, $job, $data)`。

通常在 AppServiceProvider 类的 boot() 方法中处理队列回调，其中失败事件可以直接在任务类中重写 failed() 方法来处理任务运行失败后的操作。

这里的 $connection 指的是队列连接类型，比如 database。

`$job` 是指当前任务实例本身的完整信息，其对应的类定义是：Illuminate\Queue\Jobs\DatabaseJob，其中有很多方法可以获取当前队列和任务对象的信息。

`$data` 是被序列化的 `$job` 对象属性对象，被存储在任务表的 payload 字段，可以通过 unserialize($data['data']['command']) 来恢复该实例对象。

- 「侦听」任务

代码逻辑写好之后，只是单纯将任务入队操作，并不会执行。需要启动 Laravel 侦听器来处理这些队列任务：

```
    php artisan queue:listen --queue=default,capture
```

`--queue=default,capture` 中代表此侦听器将处理哪些队列，排越靠左的队列，优先级越高。

## 释放和删除

从队列中释放一个任务，只是把这个任务重新放回队列，并不是从队列中删除。

要删除一个队列，需要删除队列中相应的记录，比如是数据库驱动的队列，就需要删除任务表中相关的记录。

也可以用 Artisan 命令来删除失败任务:

```
    php artisan queue:forget 5    # 删除掉某个失败任务
     
    php artisan queue:flush    # 删除所有失败任务
```

非失败任务不能删除，因为任务成功后会自动删除。


## Laravel 队列线上环境实践

在实际应用（线上）中，往往不是使用 queue:listen 来处理队列任务，而是通过 queue:work 配合 --daemon 选项，再加上 supervisor 进程管理器，来处理线上环境的队列任务。

相关配置及脚本如下：

- 队列任务批量部署脚本
```
    #! /bin/env bash
    # deploy.sh
    workers=1
    worker_len=16
    sleep_secs=3
    #path=/data/wwwroot/eme.dev
    path=/data/wwwroot/www.eme168.com
    while (( $workers<=$worker_len ))
    do
            echo "deploying phjs-cc queue daemon... ($workers/$worker_len)"
            php $path/artisan queue:work database --queue=phjs-cc-$workers --memory=256 --sleep=2 --tries=2 --daemon &
            sleep $sleep_secs
            echo "deploying shrinking-thumb queue daemon... ($workers/$worker_len)"
            php $path/artisan queue:work database --queue=shrink-thumb-$workers --memory=256 --sleep=2 --tries=2 --daemon &
            sleep $sleep_secs
            echo "deploying qn-upload queue daemon... ($workers/$worker_len)"
            php $path/artisan queue:work database --queue=qn-upload-$workers --memory=128 --sleep=2 --tries=2 --daemon &
            sleep $sleep_secs
            let "workers++"
    done
    echo 'deploy finished.'
    exit 0
    #! /bin/env bash
    # undeploy.sh
    workers=0
    worker_len=16
    sleep_secs=1
    while (( $workers<=$worker_len ))
    do
            echo "un-deploying phjs-cc queue daemon... ($workers/$worker_len)"
            `tmux kill-session -t "phjs-cc-$workers"`
            sleep $sleep_secs
            echo "un-deploying shrinking-thumb queue daemon... ($workers/$worker_len)"
            `tmux kill-session -t "shrink-thumb-$workers"`
            sleep $sleep_secs
            echo "un-deploying qn-upload queue daemon... ($workers/$worker_len)"
            `tmux kill-session -t "qn-upload-$workers"`
            sleep $sleep_secs
            let "workers++"
    done
    echo 'un-deploy finished.'
    exit 0
    # ps -ef | grep php | grep -v grep | awk '{print $2}' | xargs kill -9
```

这里部署的后台处理进程个数根据服务器配置情况而定，这里是在 4 核／16G／20M 的阿里云 ECS 机器上部署的。

 - 安装&使用 supervisor
 
 ```
    # 以 CentOS 6 为例
    yum info supervisor
    yum install -y supervisor
    service supervisord start|stop|restart
    supervisorctl reread
    supervisorctl start proj-worker:*
    vim /etc/supervisord.conf
 ```
 
 - supervisor 对应配置
 
 ```
    [program:proj-phjs]
    process_name=%(program_name)s_%(process_num)02d
    command=php /data/wwwroot/www.proj.com/artisan queue:work database --queue=phjs-cc-%(process_num)2d --memory=256 --sleep=2 --tries=2 --daemon
    autostart=true
    autorestart=true
    user=www
    numprocs=16
    redirect_stderr=true
    stdout_logfile=/data/wwwroot/www.proj.com/storage/logs/sv-worker.log
    [program:proj-shrink]
    process_name=%(program_name)s_%(process_num)02d
    command=php /data/wwwroot/www.proj.com/artisan queue:work database --queue=shrink-thumb-%(process_num)2d --memory=256 --sleep=2 --tries=2 --daemon
    autostart=true
    autorestart=true
    user=www
    numprocs=16
    redirect_stderr=true
    stdout_logfile=/data/wwwroot/www.proj.com/storage/logs/sv-worker.log
    [program:proj-upload]
    process_name=%(program_name)s_%(process_num)02d
    command=php /data/wwwroot/www.proj.com/artisan queue:work database --queue=qn-upload-%(process_num)2d --memory=256 --sleep=2 --tries=2 --daemon
    autostart=true
    autorestart=true
    user=www
    numprocs=16
    redirect_stderr=true
    stdout_logfile=/data/wwwroot/www.proj.com/storage/logs/sv-worker.log
 ```
 
 注意这里的 numprocs 数最好和 deploy.sh 里面的 workers 数量保持一致，因为确保每个后台运行的队列处理进程都能被 supervisor 监控到。
 
 其中，%(process_num)2d 对应的是数 numprocs 内的序号。0 表示不足多少位（这里是2位）时填充。
 

# 其他

## Composer

## 安装指定版本的 Laravel

```
    composer create-project laravel/laravel laravel.dev 5.1
```

## `composer create-project` 和 `laravel new` 安装方式的区别？

前者会通过 Composer 的机制解析并获得 Laravel 的下载包地址后然后下载，再加上国内 packgist.org 的速度不行，不换源很可能打不开，因此导致很慢。

而后者是直接获得 cabinet.laravel.com/latest.zip，自然速度快很多。

## dist/source/auto 的区别

通过 Composer 下载的软件包，是由软件包的「名字+版本号」唯一定义的，实际上，Composer 会把每个版本都当作一个独立的软件包，这个差别在从使用 Composer 的角度来说是不关紧要的，但是当你想要改变源代码的时候就变的很重要了。

不过，除了「名字」+「版本号」之外，还有「元数据」。

> The information most relevant for installation is the source definition, which describes where to get the package contents. The package data points to the contents of the package. And there are two options here: dist and source.

- `dist`

The dist is a packaged version of the package data. Usually a released version, usually a stable release.

- `source`
The source is used for development. This will usually originate from a source code repository, such as git. You can fetch this when you want to modify the downloaded package.

在安装软件包的时候，dist 和 source 选项可以同时指定，默认是 auto。


# FAQ

- **[Doctrine\DBAL\DBALException] Unknown database type enum requested, Doctrine\DBAL\Platforms\MySqlPlatform may not support**

在含有 enum 属性字段的时候，如果进行重命名等操作，出现了次错误，有一个 workaround 可以解决，需要在具体的修改操作之前加上下面这段代码：

```
    ### 添加的代码 start
    Schema::getConnection()
      ->getDoctrineSchemaManager()
      ->getDatabasePlatform()
      ->registerDoctrineTypeMapping('enum', 'string');
    ### 添加的代码 end
    ### 重命名操作
    Schema::table('users', function (Blueprint $table) {
      $table->renameColumn('password', 'u_passwd');
    });
```

> 详细的讨论见：<u>https://github.com/laravel/framework/issues/1186</u>

- 明明存在类却提示找不到类？

检查是否更改过类文件名，如果改过，则执行：`composer dump-autoload` 即可修复。

- **Class ‘Doctrine\DBAL\Driver\PDOMySql\Driver’ not found**

在确认 composer.json 中 require 字段中含有 "doctrine/dbal": "~2.3" 依赖后，重新执行下安装命令：

```
    composer require doctrine/dba
```

- session flash 无效？
只在本此请求中有效。

- **Composer：Could not find package * at any version for your minimum-stability (stable).**
> Check the package spelling or your min imum-stability: <u>https://github.com/composer/composer/issues/5118</u>

- **Composer：Composer install requires dev minimum-stability**
> <u>https://github.com/BGSU-LITS/learning-tools/issues/1</u>


# 参考


 - [Laravel 5.1 LTS 中文手册 & Laravel 5.1 LTS Documentation](http://d.laravel-china.org/docs/5.1/)
 - [大批量假数据填充的正确方法](大批量假数据填充的正确方法)