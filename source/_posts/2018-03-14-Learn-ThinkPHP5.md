---
title: ThinkPHP5 「学习笔记」
date: 2018-03-14 10:14:04
categories: ThinkPHP5
tags: [ThinkPHP5, Framework]
---

很久以前一直用`ThinkPHP 3.2`的版本，又很长时间没有使用过`ThinkPHP`，使用`Laravel`、`Lumen`很久，所以需要重新学习下`ThinkPHP5`

<!-- more -->

# 基础

## 安装

```
    composer create-project topthink/think=5.0.* tp5 --prefer-dist
```

## 目录结构

```
    project  应用部署目录
    ├─application           应用目录（可设置）
    │  ├─common             公共模块目录（可更改）
    │  ├─index              模块目录(可更改)
    │  │  ├─config.php      模块配置文件
    │  │  ├─common.php      模块函数文件
    │  │  ├─controller      控制器目录
    │  │  ├─model           模型目录
    │  │  ├─view            视图目录
    │  │  └─ ...            更多类库目录
    │  ├─command.php        命令行工具配置文件
    │  ├─common.php         应用公共（函数）文件
    │  ├─config.php         应用（公共）配置文件
    │  ├─database.php       数据库配置文件
    │  ├─tags.php           应用行为扩展定义文件
    │  └─route.php          路由配置文件
    ├─extend                扩展类库目录（可定义）
    ├─public                WEB 部署目录（对外访问目录）
    │  ├─static             静态资源存放目录(css,js,image)
    │  ├─index.php          应用入口文件
    │  ├─router.php         快速测试文件
    │  └─.htaccess          用于 apache 的重写
    ├─runtime               应用的运行时目录（可写，可设置）
    ├─vendor                第三方类库目录（Composer）
    ├─thinkphp              框架系统目录
    │  ├─lang               语言包目录
    │  ├─library            框架核心类库目录
    │  │  ├─think           Think 类库包目录
    │  │  └─traits          系统 Traits 目录
    │  ├─tpl                系统模板目录
    │  ├─.htaccess          用于 apache 的重写
    │  ├─.travis.yml        CI 定义文件
    │  ├─base.php           基础定义文件
    │  ├─composer.json      composer 定义文件
    │  ├─console.php        控制台入口文件
    │  ├─convention.php     惯例配置文件
    │  ├─helper.php         助手函数文件（可选）
    │  ├─LICENSE.txt        授权说明文件
    │  ├─phpunit.xml        单元测试配置文件
    │  ├─README.md          README 文件
    │  └─start.php          框架引导文件
    ├─build.php             自动生成定义文件（参考）
    ├─composer.json         composer 定义文件
    ├─LICENSE.txt           授权说明文件
    ├─README.md             README 文件
    ├─think                 命令行入口文件
```

> 如果是mac或者linux环境，请确保runtime目录有可写权限

```
    chmod -R 777 runtime/
```


# 架构

## 路由访问

5.0的URL访问必须是PATH_INFO方式（包括兼容方式）的URL地址，例如：

```
    http://serverName/index.php（或者其它应用入口文件）/模块/控制器/操作/参数/值…
```

### URL大小写

```php
    http://localhost/index.php/Index/BlogTest/read
    // 和下面的访问是等效的
    http://localhost/index.php/index/blogtest/read
```

如果希望URL访问严格区分大小写，可以在应用配置文件中设置：

```php
    // 关闭URL中控制器和操作名的自动转换
    'url_convert'    =>  false,
```

## 入口文件

**index.php**

```
    // 应用入口文件

    // 定义项目路径
    define('APP_PATH', __DIR__ . '/../application/');
    // 加载框架引导文件
    require __DIR__ . '/../thinkphp/start.php';
```

### 隐藏入口文件

**Apache**:

- 1、`httpd.conf`配置文件中加载了`mod_rewrite.so`模块
- 2、`AllowOverride None` 将`None`改为 `All`
- 3、在应用入口文件同级目录添加`.htaccess`文件，内容如下：

```
    <IfModule mod_rewrite.c>
    Options +FollowSymlinks -Multiviews
    RewriteEngine on

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ index.php/$1 [QSA,PT,L]
    </IfModule>
```


## 命名空间

特别注意的是，如果你需要调用PHP内置的类库，或者第三方没有使用命名空间的类库，记得在实例化类库的时候加上 \，例如：

```
    // 错误的用法
    $class = new stdClass();
    $xml  =  new SimpleXmlElement($xmlstr);

    // 正确的用法
    $class = new \stdClass();
    $xml  =  new \SimpleXmlElement($xmlstr);
```

### 根命名空间（类库包）

|名称|描述|类库目录|
|:----:|:----:|:----:|
|think|系统核心类库|thinkphp/library/think|
|traits|系统Trait类库|thinkphp/library/traits|
|app|应用类库|application|


## API

### 数据输出

**默认JSONf输出**

```php
    'default_return_type'=>'json'
```

#### JSON

```php
    namespace app\index\controller;

    class Index 
    {
        public function index()
        {
            $data = ['name'=>'thinkphp','url'=>'thinkphp.cn'];

            // 指定json数据输出
            return json([
                'data'    => $data,
                'code'    => 1,
                'message' => '操作完成'
            ]);
        }
    }   
```

#### XML

```php
    namespace app\index\controller;

    class Index 
    {
        public function index()
        {
            $data = ['name'=>'thinkphp','url'=>'thinkphp.cn'];
            // 指定xml数据输出
            return xml([
                'data'    => $data,
                'code'    => 1,
                'message' => '操作完成'
            ]);
        }
    }
```

> 核心支持的数据类型包括view、xml、json和jsonp，其他类型的需要自己扩展。


### 错误调试

设置方式：

```php
    'app_trace' => true,
    'trace'     => [
        'type'             => 'socket', 
        // socket服务器
        'host'             => 'slog.thinkphp.cn',
    ],
```


# 配置

## 默认配置定义格式

```php
    //项目配置文件
    return [
        // 默认模块名
        'default_module'        => 'index',
        // 默认控制器名
        'default_controller'    => 'Index',
        // 默认操作名
        'default_action'        => 'index',
        //更多配置参数
        //...
    ];
```

默认方式为PHP数组方式定义配置文件，你可以在入口文件定义CONF_EXT常量来更改为其它的配置类型：

```php
    // 更改配置格式为ini格式
    define('CONF_EXT', '.ini');
```

### ini格式配置示例：

```php
    default_module=Index ;默认模块
    default_controller=index ;默认控制器
    default_action=index ;默认操作
```

### xml格式配置示例：

```php
    <config>
    <default_module>Index</default_module>
    <default_controller>index</default_controller>
    <default_action>index</default_action>
    </config>
```

### json格式配置示例：

```php
    {
    "default_module":"Index",
    "default_controller":"index",
    "default_action":"index"
    }
```


## 二级配置

```php
    $config = [
        'user'  =>  [
            'type'  =>  1,
            'name'  =>  'thinkphp',
        ],
        'db'    =>  [
            'type'      =>  'mysql',
            'user'      =>  'root',
            'password'  =>  '',
        ],
    ];
    // 设置配置参数
    Config::set($config);
    // 读取二级配置参数
    echo Config::get('user.type');
    // 或者使用助手函数
    echo config('user.type');
```

## 配置加载

应用配置文件是应用初始化的时候首先加载的公共配置文件，默认位于`application/config.php`

### 扩展配置

扩展配置文件是由`extra_config_list`配置参数定义的额外的配置文件，默认会加载`database`和`validate`两个扩展配置文件。

> V5.0.1开始，取消了该配置参数，扩展配置文件直接放入application/extra目录会自动加载

### 场景配置

举个例子，你需要在公司和家里分别设置不同的数据库测试环境。那么可以这样处理，在公司环境中，我们在应用配置文件中配置：

```php
    'app_status'=>'office'
```

那么就会自动加载该状态对应的配置文件（默认位于`application/office.php`）

如果我们回家后，我们修改定义为：

```php
    'app_status'=>'home'
```

那么就会自动加载该状态对应的配置文件（位于`application/home.php`）。

## 读取配置

```php
    echo Config::get('配置参数1');

    echo config('配置参数1');

    // 读取所有的配置参数
    dump(Config::get());
    // Or
    dump(config());

    // 判定是否存在某个设置参数
    Config::has('配置参数2');
    // Or
    config('?配置参数2');

    // 读取二级配置
    echo Config::get('配置参数.二级参数');
    echo config('配置参数.二级参数');

```

### 设置配置参数

```php
    Config::set('配置参数', '配置值');
    config('配置参数'，'配置值');

    // 批量设置
    Config::set([
        '配置参数1' => '配置值',
        '配置参数2' => '配置值',
    ]);
    // 或者使用助手函数
    config([
        '配置参数1'=>'配置值',
        '配置参数2'=>'配置值'
    ]);
```

## 配置作用域

```php
    // 导入my_config.php中的配置参数，并纳入user作用域
    Config::load('my_config.php','','user'); 

    // 解析并导入my_config.ini 中的配置参数，读入test作用域
    Config::parse('my_config.ini','ini','test'); 

    // 设置user_type参数，并纳入user作用域
    Config::set('user_type',1,'user'); 

    // 批量设置配置参数，并纳入test作用域
    Config::set($config,'test'); 

    // 读取user作用域的user_type配置参数
    echo Config::get('user_type','user'); 

    // 读取user作用域下面的所有配置参数
    dump(Config::get('','user')); 
    dump(config('',null,'user')); // 同上

    // 判断在test作用域下面是否存在user_type参数
    Config::has('user_type','test'); 
```

可以使用range方法切换当前配置文件的作用域，例如：

```php
    Config::range('test');
```

## 环境变量配置

在开发过程中，可以在应用根目录下面的`.env`来模拟环境变量配置，`.env`文件中的配置参数定义格式采用`ini`方式，例如：

```php
    app_debug =  true
    app_trace =  true
```

如果你的部署环境单独配置了环境变量，那么请删除`.env`配置文件，避免冲突。

环境变量配置的参数会全部转换为大写，值为 `null`，`no` 和 `false` 等效于 `""`，值为 `yes` 和 `true` 等效于 `"1"`。

注意，环境变量不支持数组参数，如果需要使用数组参数可以，使用下划线分割定义配置参数名：

```php
    database_username =  root
    database_password =  123456

    // 或者
    [database]
    username =  root
    password =  123456
```

### 获取环境变量

```php
    Env::get('database.username');
    Env::get('database.password');

    // 同时下面的方式也可以获取
    Env::get('database_username');
    Env::get('database_password');

    // 可以支持默认值
    // 获取环境变量 如果不存在则使用默认值root
    Env::get('database.username','root');

    // 直接在应用配置中使用环境变量
    return [
        'hostname'  =>  Env::get('hostname','127.0.0.1'),
    ];
```

# 路由

## 路由模式

### 普通模式

```php
    'url_route_on'  =>  false,
```

路由关闭后，不会解析任何路由规则，采用默认的PATH_INFO 模式访问URL：

```php
    http://serverName/index.php/module/controller/action/param/value/...
```

### 混合模式

开启路由，并使用路由定义+默认PATH_INFO方式的混合：

```php
    'url_route_on'  =>  true,
    'url_route_must'=>  false,
```

### 强制模式

开启路由，并设置必须定义路由才能访问：

```php
    'url_route_on'          =>  true,
    'url_route_must'        =>  true,
```

这种方式下面必须严格给每一个访问地址定义路由规则（包括首页），否则将抛出异常。

首页的路由规则采用`/`定义即可，例如下面把网站首页路由输出`Hello,world!`

```php
    Route::get('/', function () {
        return 'Hello TP5';
    });
```

## 路由定义

### 动态注册

路由定义采用`\think\Route`类的`rule`方法注册，通常是在应用的路由配置文件`application/route.php`进行注册，格式是：

> Route::rule('路由表达式','路由地址','请求类型','路由参数（数组）','变量规则（数组）');

```php
    use think\Route;
    // 注册路由到index模块的News控制器的read操作
    Route::rule('new/:id','index/News/read');
```

系统提供了为不同的请求类型定义路由规则的简化方法，例如：

```php
    Route::get('new/:id','News/read'); // 定义GET请求路由规则
    Route::post('new/:id','News/update'); // 定义POST请求路由规则
    Route::put('new/:id','News/update'); // 定义PUT请求路由规则
    Route::delete('new/:id','News/delete'); // 定义DELETE请求路由规则
    Route::any('new/:id','News/read'); // 所有请求都支持的路由规则
```

如果要定义get和post请求支持的路由规则，也可以用：

```php
    Route::rule('new/:id','News/read','GET|POST');
```

### 规则表达式

规则表达式通常包含静态地址和动态地址，或者两种地址的结合，例如下面都属于有效的规则表达式：

```php
    '/' => 'index', // 首页访问路由
    'my' => 'Member/myinfo', // 静态地址路由
    'blog/:id' => 'Blog/read', // 静态地址和动态地址结合
    'new/:year/:month/:day' => 'News/read', // 静态地址和动态地址结合
    ':user/:blog_id' => 'Blog/read',// 全动态地址
```

## 批量注册路由

```php
    // 批量注册GET路由
    Route::get([
        'new/:id'  =>  'News/read',
        'blog/:id' =>  ['Blog/edit',[],['id'=>'\d+']]
        ...
    ]);

    // 效果等同于
    Route::rule([
        'new/:id'  =>  'News/read',
        'blog/:id' =>  ['Blog/edit',[],['id'=>'\d+']]
        ...
    ],'','GET');
```

### 定义路由配置文件

路由动态注册和配置定义的方式可以共存，例如：

```php
    use think\Route;

    Route::rule('hello/:name','index/index/hello');

    return [
        'new/:id'   => 'News/read',
        'blog/:id'   => ['Blog/update',['method' => 'post|put'], ['id' => '\d+']],
    ];
```

默认情况下，只会加载一个路由配置文件`route.php`，如果你需要定义多个路由文件，可以修改`route_config_file`配置参数，例如：

```php
    // 定义路由配置文件（数组）
    'route_config_file' =>  ['route', 'route1', 'route2'],
```

## 变量规则

### 全局变量规则

```php
    // 设置name变量规则（采用正则定义）
    Route::pattern('name','\w+');

    // 支持批量添加
    Route::pattern([
        'name'  =>  '\w+',
        'id'    =>  '\d+',
    ]);
```

### 局部变量规则

局部变量规则，仅在当前路由有效：

```php
    // 定义GET请求路由规则 并设置name变量规则
    Route::get('new/:name','News/read',[],['name'=>'\w+']);
```

> 如果一个变量同时定义了全局规则和局部规则，局部规则会覆盖全局变量的定义

### 完整URL规则

如果要对整个URL进行规则检查，可以进行`__url__` 变量规则，例如：

```php
    // 定义GET请求路由规则 并设置完整URL变量规则
    Route::get('new/:id','News/read',[],['__url__'=>'new\/\w+$']);
```

## 资源路由

5.0支持设置`RESTFul`请求的资源路由，方式如下：

```php
    Route::resource('blog','index/blog');
```

或者在路由配置文件中使用`__rest__`添加资源路由定义：

```php
    return [
        // 定义资源路由
        '__rest__'=>[
            // 指向index模块的blog控制器
            'blog'=>'index/blog',
        ],
        // 定义普通路由
        'hello/:id'=>'index/hello',
    ]
```

设置后会自动注册`7`个路由规则，如下：

|标识|请求类型|生成路由规则|对应操作|
|:----:|:----:|:----:|:----:|
|index|GET|`blog`|index|
|create|GET|`blog/create`|create|
|save|POST|`blog`|save|
|read|GET|`blog/:id`|read|
|edit|GET|`blog/:id/edit`|edit|
|update|PUT|`blog/:id`|update|
|delete|DELETE|`blog/:id`|delete|

也可以在定义资源路由的时候限定执行的方法（标识），例如：

```php
    // 只允许index read edit update 四个操作
    Route::resource('blog','index/blog',['only'=>['index','read','edit','update']]);
    // 排除index和delete操作
    Route::resource('blog','index/blog',['except'=>['index','delete']]);
```

如果需要更改某个资源路由标识的对应操作，可以使用下面方法:

```php
    Route::rest('create', ['GET', '/add', 'add']);
```

设置之后，URL访问变为：

```php
    http://serverName/blog/create
    变成
    http://serverName/blog/add
```

支持批量更改，如下：

```php
    Route::rest([
        'save'   => ['POST', '', 'store'],
        'update' => ['PUT', '/:id', 'save'],
        'delete' => ['DELETE', '/:id', 'destory'],
    ]);
```

## 快捷路由

```php
    // 给User控制器设置快捷路由
    Route::controller('user','index/User');
```

User控制器定义如下：

```php
    namespace app\index\controller;

    class User {
        public function getInfo()
        {
        }
        
        public function getPhone()
        {
        }
        
        public function postInfo()
        {
        }
        
        public function putInfo()
        {
        }
        
        public function deleteInfo()
        {
        }
    }
```

我们可以通过下面的URL访问

```php
    get http://localhost/user/info
    get http://localhost/user/phone
    post http://localhost/user/info
    put http://localhost/user/info
    delete http://localhost/user/info
```

## 路由别名

```php
    // user 别名路由到 index/User 控制器
    Route::alias('user','index/User');
```

如果在路由配置文件`route.php`中定义的话，使用：

```php
    return [
        '__alias__' =>  [
            'user'  =>  'index/User',
        ],
    ];
```

## 闭包支持

闭包定义的时候支持参数传递，例如：

```php
    Route::get('hello/:name',function($name){ 
        return 'Hello,'.$name;
    });
```

## 绑定模型

```php
    Route::rule('hello/:id','index/index/hello','GET',[
        'ext'           =>  'html',
        'bind_model'    =>  [
        'user'          =>  function($param){
            $model = new \app\index\model\User;

            return $model->where($param)->find();
        }
        ],
    ]);
```

闭包函数的参数就是当前请求的URL变量信息。

在控制器中可以通过下面的代码或者使用依赖注入获取：

```php
    request()->user;
```

# 控制器

## 渲染输出

```php
    namespace app\index\controller;

    class Index
    {
        public function hello()
        {
            return 'hello world!';
        }

        public function json()
        {
            return json_encode($data);
        }

        public function read()
        {
            return view();
        }
    }
```

## 跳转和重定向

```php
    public function index()
    {
        $User = new User;
        $result = $User->save($data);

        if ($result) {
            $this->success('新增成功', 'User/list');
        } else {
            $this->error('新增失败');
        }
    }
```

跳转地址是可选的，success方法的默认跳转地址是`$_SERVER["HTTP_REFERER"]`，error方法的默认跳转地址是`javascript:history.back(-1)`;。

> 默认的等待时间都是3秒

`success`和`error`方法都可以对应的模板，默认的设置是两个方法对应的模板都是：

```php
    THINK_PATH . 'tpl/dispatch_jump.tpl'
```

改变默认的模板：

```php
    //默认错误跳转对应的模板文件
    'dispatch_error_tmpl' => APP_PATH . 'tpl/dispatch_jump.tpl',
    //默认成功跳转对应的模板文件
    'dispatch_success_tmpl' => APP_PATH . 'tpl/dispatch_jump.tpl',
```

也可以使用项目内部的模板文件

```php
    //默认错误跳转对应的模板文件
    'dispatch_error_tmpl' => 'public/error',
    //默认成功跳转对应的模板文件
    'dispatch_success_tmpl' => 'public/success',
```

### 重定向

```php
    //重定向到News模块的Category操作
    $this->redirect('News/category', ['cate_id' => 2]);

    //重定向到指定的URL地址 并且使用302
    $this->redirect('http://thinkphp.cn/blog/2',302);

    // 通过session闪存数据传值
    $this->redirect('News/category', ['cate_id' => 2], 302, ['data' => 'hello'])

    // 记住当前的URL后跳转
    redirect('News/category')->remember();

    // 需要跳转到上次记住的URL的时候使用：
    redirect()->restore();
```


## Rest控制器

### RESTFul方法定义

```php
    <?php

    namespace app\index\controller;

    use think\controller\Rest;

    class Blog extends Rest
    {
        public function rest()
        {
            switch ($this->method){
            case 'get': // get请求处理代码
                if ($this->type == 'html'){
                } elseif ($this->type == 'xml'){
                }
                break;
            case 'put': // put请求处理代码
                break;
            case 'post': // post请求处理代码
                break;
            }
        }
    }
```

在Rest操作方法中，可以使用`$this->type`获取当前访问的资源类型，用`$this->method`获取当前的请求类型

### RESTFul 输出

#### 使用Rest类提供的 `response` 方法

```php
    $this->response($data, 'json', 200);
```

#### 使用 `think\Response` 类

```php
    Response::create($data, 'json')->code(200);
```

#### 使用助手函数

```php
json($data, 200);
```

$data为需要输出的数据，第二个参数为输出数据的http状态码

```php
    // 输出 json 格式数据
    json($data, 200);

    // 输出 jsonp 格式数据
    jsonp($data, 200);

    // 输出 xml 格式数据
    xml($data, 200);
```


## 资源控制器

资源控制器可以让你轻松的创建RESTFul资源控制器，可以通过命令行生成需要的资源控制器，例如：

```php
    // 生成index模块的Blog资源控制器
    php think make:controller index/Blog

    // 或者使用完整的命名空间生成
    php think make:controller app\index\controller\Blog
```

然后你只需要为资源控制器注册一个资源路由：

```php
    Route::resource('blog', 'index/Blog');
```

设置后会自动注册7个路由规则，如下：

|请求类型|生成路由规则|对应操作方法|
|:----|:----|:----|
|GET|`blog`|index|
|GET|`blog/create`|create|
|POST|`blog`|save|
|GET|`blog/:id`|read|
|GET|`blog/:id/edit`|edit|
|PUT|`blog/:id`|update|
|DELETE|`blog/:id`|delete|


# 请求

## 请求信息

如果要获取当前的请求信息，可以使用\think\Request类，
除了下文中的：

```php
    $request = Request::instance();

    // 助手函数
    $request = request();
```

### 获取URL信息

```php
    $request = Request::instance();

    // 获取当前域名
    echo 'domain: ' . $request->domain() . '<br/>';

    // 获取当前入口文件
    echo 'file: ' . $request->baseFile() . '<br/>';

    // 获取当前URL地址 不含域名
    echo 'url: ' . $request->url() . '<br/>';

    // 获取包含域名的完整URL地址
    echo 'url with domain: ' . $request->url(true) . '<br/>';

    // 获取当前URL地址 不含QUERY_STRING
    echo 'url without query: ' . $request->baseUrl() . '<br/>';

    // 获取URL访问的ROOT地址
    echo 'root:' . $request->root() . '<br/>';

    // 获取URL访问的ROOT地址
    echo 'root with domain: ' . $request->root(true) . '<br/>';

    // 获取URL地址中的PATH_INFO信息
    echo 'pathinfo: ' . $request->pathinfo() . '<br/>';

    // 获取URL地址中的PATH_INFO信息 不含后缀
    echo 'pathinfo: ' . $request->path() . '<br/>';

    // 获取URL地址中的后缀信息
    echo 'ext: ' . $request->ext() . '<br/>';

```


### 设置/获取 模块/控制器/操作名称

```php
    $request = Request::instance();
    echo "当前模块名称是" . $request->module();
    echo "当前控制器名称是" . $request->controller();
    echo "当前操作名称是" . $request->action();
```


### 获取请求参数

```php
    $request = Request::instance();
    echo '请求方法：' . $request->method() . '<br/>';
    echo '资源类型：' . $request->type() . '<br/>';
    echo '访问ip地址：' . $request->ip() . '<br/>';
    echo '是否AJax请求：' . var_export($request->isAjax(), true) . '<br/>';
    echo '请求参数：';
    dump($request->param());
    echo '请求参数：仅包含name';
    dump($request->only(['name']));
    echo '请求参数：排除name';
    dump($request->except(['name']));
```

### 获取路由和调度信息

```php
    $request = Request::instance();
    echo '路由信息：';
    dump($request->route());
    echo '调度信息：';
    dump($request->dispatch());
```


### 设置请求信息

```php
    $request = Request::instance();
    $request->root('index.php');
    $request->pathinfo('index/index/hello');
```


## 输入变量

可以通过`Request`对象完成全局输入变量的检测、获取和安全过滤，支持包括`$_GET`、`$_POST`、`$_REQUEST`、`$_SERVER`、`$_SESSION`、`$_COOKIE`、`$_ENV`等系统变量，以及文件上传信息


### 检测变量是否设置

```php
    Request::instance()->has('id','get');
    Request::instance()->has('name','post');

    // 助手函数
    input('?get.id');
    input('?post.name');
```

### 获取`PARAM`变量

```php
    // 获取当前请求的name变量
    Request::instance()->param('name');
    // 获取当前请求的所有变量（经过过滤）
    Request::instance()->param();
    // 获取当前请求的所有变量（原始数据）
    Request::instance()->param(false);
    // 获取当前请求的所有变量（包含上传文件）
    Request::instance()->param(true);

    // 助手函数
    input('param.name');
    input('param.');
    // 或者
    input('name');
    input('');
```

### 获取`GET`变量

```php
    Request::instance()->get('id'); // 获取某个get变量
    Request::instance()->get('name'); // 获取get变量
    Request::instance()->get(); // 获取所有的get变量（经过过滤的数组）
    Request::instance()->get(false); // 获取所有的get变量（原始数组）

    // 助手函数
    input('get.id');
    input('get.name');
    input('get.');
```

### 获取`POST`变量

```php
    Request::instance()->post('name'); // 获取某个post变量
    Request::instance()->post(); // 获取经过过滤的全部post变量
    Request::instance()->post(false); // 获取全部的post原始变量

    // 助手函数
    input('post.name');
    input('post.');
```

。。。其他几个类似。。。


## 变量过滤

```php
    // 默认全局过滤方法 用逗号分隔多个
    'default_filter'         => 'htmlspecialchars',
```

设置全局变量过滤方法：

```php
    Request::instance()->filter('htmlspecialchars');
```

支持设置多个过滤方法，例如：

```php
    Request::instance()->filter(['strip_tags','htmlspecialchars']),
```

也可以在获取变量的时候添加过滤方法，例如：

```php
    Request::instance()->get('name','','htmlspecialchars'); // 获取get变量 并用htmlspecialchars函数过滤
    Request::instance()->param('username','','strip_tags'); // 获取param变量 并用strip_tags函数过滤
    Request::instance()->post('name','','org\Filter::safeHtml'); // 获取post变量 并用org\Filter类的safeHtml方法过滤
```


## 获取部分变量

```php
    // 只获取当前请求的id和name变量
    Request::instance()->only('id,name');

    // 或者使用数组方式
    // 只获取当前请求的id和name变量
    Request::instance()->only(['id','name']);

    // 默认获取的是当前请求参数，如果需要获取其它类型的参数，可以使用第二个参数，例如
    // 只获取GET请求的id和name变量
    Request::instance()->only(['id','name'],'get');
    // 只获取POST请求的id和name变量
    Request::instance()->only(['id','name'],'post');
```

## 排除部分变量

```php
    // 排除id和name变量
    Request::instance()->except('id,name');

    // 排除id和name变量
    Request::instance()->except(['id','name']);

    // 同样支持指定变量类型获取：
    // 排除GET请求的id和name变量
    Request::instance()->except(['id','name'],'get');
    // 排除POST请求的id和name变量
    Request::instance()->except(['id','name'],'post');
```


## 请求类型

```php
    // 是否为 GET 请求
    if (Request::instance()->isGet()) echo "当前为 GET 请求";

    // 是否为 POST 请求
    if (Request::instance()->isPost()) echo "当前为 POST 请求";

    // 是否为 PUT 请求
    if (Request::instance()->isPut()) echo "当前为 PUT 请求";

    // 是否为 DELETE 请求
    if (Request::instance()->isDelete()) echo "当前为 DELETE 请求";

    // 是否为 Ajax 请求
    if (Request::instance()->isAjax()) echo "当前为 Ajax 请求";

    // 是否为 Pjax 请求
    if (Request::instance()->isPjax()) echo "当前为 Pjax 请求";

    // 是否为手机访问
    if (Request::instance()->isMobile()) echo "当前为手机访问";

    // 是否为 HEAD 请求
    if (Request::instance()->isHead()) echo "当前为 HEAD 请求";

    // 是否为 Patch 请求
    if (Request::instance()->isPatch()) echo "当前为 PATCH 请求";

    // 是否为 OPTIONS 请求
    if (Request::instance()->isOptions()) echo "当前为 OPTIONS 请求";

    // 是否为 cli
    if (Request::instance()->isCli()) echo "当前为 cli";

    // 是否为 cgi
    if (Request::instance()->isCgi()) echo "当前为 cgi";

    // 助手函数
    // 是否为 GET 请求
    if (request()->isGet()) echo "当前为 GET 请求";
```

##  请求类型伪装

支持请求类型伪装，可以在`POST`表单里面提交`_method`变量，传入需要伪装的请求类型，例如：

```php
    <form method="post" action="">
        <input type="text" name="name" value="Hello">
        <input type="hidden" name="_method" value="PUT" >
        <input type="submit" value="提交">
    </form>
```

### `AJAX`/`PJAX`伪装

```php
    http://localhost/index?_ajax=1 
    http://localhost/index?_pjax=1 

    // 改变伪装请求的变量名，可以修改应用配置文件：
    // 表单ajax伪装变量
    'var_ajax'               => '_a',
    // 表单pjax伪装变量
    'var_pjax'               => '_p',
```

## 方法注入

如果你需要在Request请求对象中添加自己的方法，可以使用Request对象的方法注入功能，例如：

```php
    // 通过hook方法注入动态方法
    Request::hook('user','getUserInfo');
```

`getUserInfo`函数定义如下

```php
    function getUserInfo(Request $request, $userId)
    {
        // 根据$userId获取用户信息
        return $info;
    }
```

接下来，我们可以直接在控制器中使用：

```php
    public function index()
    {
        $info = Request::instance()->user($userId);
    }
```


## 属性注入

```php
    // 动态绑定属性
    Request::instance()->bind('user',new User);
    // 或者使用
    Request::instance()->user = new User;
```

获取绑定的属性使用下面的方式：

```php
    Request::instance()->user;
```

如果控制器注入请求对象的话，也可以直接使用

```php
    $this->request->user;
    // 助手函数
    request->user;
```

## 依赖注入

### 架构方法注入

```php
    namespace app\index\controller;

    use think\Request;

    class Index
    {
        protected $request;
        
        public function __construct(Request $request)
        {
            $this->request = $request;
        }
        
        public function hello()
        {
            return 'Hello,' . $this->request->param('name') . '！';
        }
        
    }

```

### 操作方法注入

```php
    namespace app\index\controller;

    use think\Request;

    class Index
    {

        public function hello(Request $request)
        {
            return 'Hello,' . $request->param('name') . '！';
        }
        
    }
```


### `invoke`方法自动调用

```php
    namespace app\index\model;

    use think\Model;

    class User extends Model
    {
        public static function invoke(Request $request)
        {
            $id = $request->param('id');
            return User::get($id);
        }
    }
```


# 数据库

## 基本使用

```php
    Db::query('select * from think_user where id=?',[8]);
    Db::execute('insert into think_user (id, name) values (?, ?)',[8,'thinkphp']);

    // 支持命名占位符绑定
    Db::query('select * from think_user where id=:id',['id'=>8]);
    Db::execute('insert into think_user (id, name) values (:id, :name)',['id'=>8,'name'=>'thinkphp']);
```

## 查询构造器

### 查询数据

```php
    // 查询一个数据使用：
    // table方法必须指定完整的数据表名
    Db::table('think_user')->where('id',1)->find();
    // find 方法查询结果不存在，返回 null

    // 查询数据集使用：
    Db::table('think_user')->where('status',1)->select();

    // 助手函数
    db('user')->where('id',1)->find();
    db('user')->where('status',1)->select();

    // 使用Query对象或闭包查询
    $query = new \think\db\Query();
    $query->table('think_user')->where('status',1);
    Db::find($query);
    Db::select($query);

    // 使用闭包函数查询
    Db::select(function($query){
        $query->table('think_user')->where('status',1);
    });

    // 值和列查询
    // 返回某个字段的值
    Db::table('think_user')->where('id',1)->value('name');

    // 查询某一列的值可以用
    // 返回数组
    Db::table('think_user')->where('status',1)->column('name');
    // 指定索引
    Db::table('think_user')->where('status',1)->column('name','id');
    Db::table('think_user')->where('status',1)->column('id,name');
    // 同tp3的getField
    

    // JSON类型数据查询
    // 查询JSON类型字段 （info字段为json类型）
    Db::table('think_user')->where('info$.email','thinkphp@qq.com')->find();
```


### 添加数据

```php
    $data = ['foo' => 'bar', 'bar' => 'foo'];
    Db::table('think_user')->insert($data);
```

如果你在database.php配置文件中配置了数据库前缀(prefix)，那么可以直接使用 Db 类的 name 方法提交数据

```php
    Db::name('user')->insert($data);
```

添加数据后如果需要返回新增数据的自增主键，可以使用`getLastInsID`方法：

```php
    Db::name('user')->insert($data);
    $userId = Db::name('user')->getLastInsID();

    // 直接使用insertGetId方法新增数据并返回主键值：
    Db::name('user')->insertGetId($data);
```

#### 添加多条数据

```php
    $data = [
        ['foo' => 'bar', 'bar' => 'foo'],
        ['foo' => 'bar1', 'bar' => 'foo1'],
        ['foo' => 'bar2', 'bar' => 'foo2']
    ];
    Db::name('user')->insertAll($data);
```

#### 助手函数

```php
    // 添加单条数据
    db('user')->insert($data);

    // 添加多条数据
    db('user')->insertAll($list);
```


### 更新数据

```php
    Db::table('think_user')->where('id', 1)->update(['name' => 'thinkphp']);

    // 如果数据中包含主键，可以直接使用：
    Db::table('think_user')->update(['name' => 'thinkphp','id'=>1]);
```

如果要更新的数据需要使用SQL函数或者其它字段，可以使用下面的方式：

```php
    Db::table('think_user')
    ->where('id', 1)
    ->update([
        'login_time'  => ['exp','now()'],
        'login_times' => ['exp','login_times+1'],
    ]);
```

#### 更新某个字段的值：

```php
    Db::table('think_user')->where('id',1)->setField('name', 'thinkphp');
```

#### 自增或自减一个字段的值

```php
    // score 字段加 1
    Db::table('think_user')->where('id', 1)->setInc('score');
    // score 字段加 5
    Db::table('think_user')->where('id', 1)->setInc('score', 5);
    // score 字段减 1
    Db::table('think_user')->where('id', 1)->setDec('score');
    // score 字段减 5
    Db::table('think_user')->where('id', 1)->setDec('score', 5);

    // 延迟更新
    Db::table('think_user')->where('id', 1)->setInc('score', 1, 10);
```

#### 助手函数

```php
    // 更新数据表中的数据
    db('user')->where('id',1)->update(['name' => 'thinkphp']);
    // 更新某个字段的值
    db('user')->where('id',1)->setField('name','thinkphp');
    // 自增 score 字段
    db('user')->where('id', 1)->setInc('score');
    // 自减 score 字段
    db('user')->where('id', 1)->setDec('score');
```

#### 链式操作方法

```php
    Db::table('data')
        ->where('id',1)
        ->inc('read')
        ->dec('score',3)
        ->exp('name','UPPER(name)')
        ->update();
```


### 删除数据

```php
    // 根据主键删除
    Db::table('think_user')->delete(1);
    Db::table('think_user')->delete([1,2,3]);

    // 条件删除    
    Db::table('think_user')->where('id',1)->delete();
    Db::table('think_user')->where('id','<',10)->delete();
```

> delete 方法返回影响数据的条数，没有删除返回 0

#### 助手函数

```php
    // 根据主键删除
    db('user')->delete(1);
    // 条件删除    
    db('user')->where('id',1)->delete();
```


### 查询方法

```php
    Db::table('think_user')
        ->where('name','like','%thinkphp')
        ->whereOr('title','like','%thinkphp')
        ->find();
```

#### 混合查询

```php
    $result = Db::table('think_user')->where(function ($query) {
    $query->where('id', 1)->whereor('id', 2);
    })->whereOr(function ($query) {
        $query->where('name', 'like', 'think')->whereOr('name', 'like', 'thinkphp');
    })->select();
```

#### `getTableInfo`方法

```php
    // 获取`think_user`表所有信息
    Db::getTableInfo('think_user');
    // 获取`think_user`表所有字段
    Db::getTableInfo('think_user', 'fields');
    // 获取`think_user`表所有字段的类型
    Db::getTableInfo('think_user', 'type');
    // 获取`think_user`表的主键
    Db::getTableInfo('think_user', 'pk');
```


### 链式操作

```php
    Db::table('think_user')
        ->where('status',1)
        ->order('create_time')
        ->limit(10)
```

### 时间查询

#### 使用`where`方法

```php
    // 大于某个时间
    where('create_time','> time','2016-1-1');
    // 小于某个时间
    where('create_time','<= time','2016-1-1');
    // 时间区间查询
    where('create_time','between time',['2015-1-1','2016-1-1']);
```

#### 使用`whereTime`方法

```php
    // 大于某个时间
    Db::table('think_user')->whereTime('birthday', '>=', '1970-10-1')->select();
    // 小于某个时间
    Db::table('think_user')->whereTime('birthday', '<', '2000-10-1')->select();
    // 时间区间查询
    Db::table('think_user')->whereTime('birthday', 'between', ['1970-10-1', '2000-10-1'])->select();
    // 不在某个时间区间
    Db::table('think_user')->whereTime('birthday', 'not between', ['1970-10-1', '2000-10-1'])->select();
```

#### 时间表达式

```php
    // 获取今天的博客
    Db::table('think_blog') ->whereTime('create_time', 'today')->select();
    // 获取昨天的博客
    Db::table('think_blog')->whereTime('create_time', 'yesterday')->select();
    // 获取本周的博客
    Db::table('think_blog')->whereTime('create_time', 'week')->select();   
    // 获取上周的博客
    Db::table('think_blog')->whereTime('create_time', 'last week')->select();    
    // 获取本月的博客
    Db::table('think_blog')->whereTime('create_time', 'month')->select();   
    // 获取上月的博客
    Db::table('think_blog')->whereTime('create_time', 'last month')->select();      
    // 获取今年的博客
    Db::table('think_blog')->whereTime('create_time', 'year')->select();    
    // 获取去年的博客
    Db::table('think_blog')->whereTime('create_time', 'last year')->select(); 
```

如果查询当天、本周、本月和今年的时间，还可以简化为：

```php
    // 获取今天的博客
    Db::table('think_blog')->whereTime('create_time', 'd')->select();
    // 获取本周的博客
    Db::table('think_blog')->whereTime('create_time', 'w')->select();   
    // 获取本月的博客
    Db::table('think_blog')->whereTime('create_time', 'm')->select();   
    // 获取今年的博客
    Db::table('think_blog')->whereTime('create_time', 'y') ->select(); 
```

```php
    // 查询两个小时内的博客
    Db::table('think_blog')->whereTime('create_time','-2 hours')->select();
```


### 快捷查询

```php
    Db::table('think_user')
        ->where('name|title','like','thinkphp%')
        ->where('create_time&update_time','>',0)
        ->find();
```

#### 区间查询

```php
    Db::table('think_user')
        ->where('name',['like','thinkphp%'],['like','%thinkphp'])
        ->where('id',['>',0],['<>',10],'or')
        ->find();
```

生成的SQL语句为：

```php
    SELECT * FROM `think_user` WHERE ( `name` LIKE 'thinkphp%' AND `name` LIKE '%thinkphp' ) AND ( `id` > 0 OR `id` <> 10 ) LIMIT 1
```


#### 批量查询

```php
    Db::table('think_user')
        ->where([
            'name'  =>  ['like','thinkphp%'],
            'title' =>  ['like','%thinkphp'],
            'id'    =>  ['>',0],
            'status'=>  1
        ])
        ->select();
```


#### 闭包查询

```php
    Db::table('think_user')->select(function($query){
        $query->where('name','thinkphp')
              ->whereOr('id','>',10);
    });
```


#### 使用`Query`对象查询

```php
    $query = new \think\db\Query;
    $query->name('user')
        ->where('name','like','%think%')
        ->where('id','>',10)
        ->limit(10);
    Db::select($query); 
```


#### 混合查询

```php
    Db::table('think_user')
        ->where('name',['like','thinkphp%'],['like','%thinkphp'])
        ->where(function($query){
            $query->where('id',['<',10],['>',100],'or');
        })
        ->select();
```

`V5.0.4+`开始，ThinkPHP支持对同一个字段多次调用查询条件，例如：

```php
    Db::table('think_user')
        ->where('name','like','%think%')
        ->where('name','like','%php%')
        ->where('id','in',[1,5,80,50])
        ->where('id','>',10)
        ->find();
```

### 子查询

#### 1、使用select方法

```php
    $subQuery = Db::table('think_user')
        ->field('id,name')
        ->where('id','>',10)
        ->select(false); 
```

#### 2、使用fetchSql方法

```php
    $subQuery = Db::table('think_user')
        ->field('id,name')
        ->where('id','>',10)
        ->fetchSql(true)
        ->select();
```

#### 3、使用buildSql构造子查询

```php
    $subQuery = Db::table('think_user')
        ->field('id,name')
        ->where('id','>',10)
        ->buildSql();
```

#### 4、使用闭包构造子查询

```php
    Db::table('think_user')
        ->where('id','IN',function($query){
            $query->table('think_profile')->where('status',1)->field('id');
        })
        ->select();
```


### 原生查询

#### `query`方法

```php
    Db::query("select * from think_user where status=1");
```

`query`方法用于执行`SQL`查询操作，如果数据非法或者查询错误则返回false，否则返回查询结果数据集（同`select`方法）。

#### `execute`方法

```php
    Db::execute("update think_user set name='thinkphp' where status=1");
```

`execute`用于更新和写入数据的sql操作，如果数据非法或者查询错误则返回false ，否则返回影响的记录数。


## 事务操作

使用事务处理的话，需要数据库引擎支持事务处理。比如 MySQL 的 `MyISAM` 不支持事务处理，需要使用 `InnoDB` 引擎。

自动控制事务处理:

```php
    Db::transaction(function(){
        Db::table('think_user')->find(1);
        Db::table('think_user')->delete(1);
    });
```

手动控制事务:

```php
    // 启动事务
    Db::startTrans();
    try{
        Db::table('think_user')->find(1);
        Db::table('think_user')->delete(1);
        // 提交事务
        Db::commit();    
    } catch (\Exception $e) {
        // 回滚事务
        Db::rollback();
    }
```


## 监听SQL

```php
    Db::listen(function($sql, $time, $explain){
        // 记录SQL
        echo $sql. ' ['.$time.'s]';
        // 查看性能分析结果
        dump($explain);
    });
```


## 分布式数据库

```php
    //分布式数据库配置定义
    return [
        // 启用分布式数据库
        'deploy'    =>  1,
        // 数据库类型
        'type'        => 'mysql',
        // 服务器地址
        'hostname'    => '192.168.1.1,192.168.1.2',
        // 数据库名
        'database'    => 'demo',
        // 数据库用户名
        'username'    => 'root',
        // 数据库密码
        'password'    => '',
        // 数据库连接端口
        'hostport'    => '',
    ]
```


# 模型

## 模型定义

```php
    // 设置主键
    protected $pk = 'uid';

    // 设置当前模型对应的完整数据表名称
    protected $table = 'think_user';
```

## 模型调用

```php
    // 静态调用
    $user = User::get(1);
    $user->name = 'thinkphp';
    $user->save();

    // 实例化模型
    $user = new User;
    $user->name= 'thinkphp';
    $user->save();

    // 使用 Loader 类实例化（单例）
    $user = Loader::model('User');

    // 或者使用助手函数`model`
    $user = model('User');
    $user->name= 'thinkphp';
    $user->save();
```

## 模型初始化

模型的初始化是重写`Model`的`initialize`

```php
    //自定义初始化
    protected function initialize()
    {
        //需要调用`Model`的`initialize`方法
        parent::initialize();
        //TODO:自定义的初始化
    }
```

也可以使用静态`init`方法

```php
    //自定义初始化
    protected static function init()
    {
        //TODO:自定义的初始化
    }
```

## CURD

### 新增

```php
    //  添加一条数据
    $user->save();

    // 过滤post数组中的非数据表字段数据
    $user->allowField(true)->save();

    // 获取自增ID
    echo $user->id;

    // 第二次开始必须使用下面的方式新增
    $user->isUpdate(false)->save();

    // 添加多条数据
    $user->saveAll($list);
```

#### 助手函数

```php
    // 使用model助手函数实例化User模型
    $user = model('User');
    // 模型对象赋值
    $user->data([
        'name'  =>  'thinkphp',
        'email' =>  'thinkphp@qq.com'
    ]);
    $user->save();

    // 批量新增
    $user = model('User');

    $list = [
        ['name'=>'thinkphp','email'=>'thinkphp@qq.com'],
        ['name'=>'onethink','email'=>'onethink@qq.com']
    ];
    $user->saveAll($list);
```

### 更新

```php
    $user = new User;
    // save方法第二个参数为更新条件
    $user->save([
        'name'  => 'thinkphp',
        'email' => 'thinkphp@qq.com'
    ],['id' => 1]);
```

### 删除

```php
    // 删除当前模型
    $user = User::get(1);
    $user->delete();

    // 根据主键删除
    User::destroy(1);
    // 支持批量删除多个数据
    User::destroy('1,2,3');
    // 或者
    User::destroy([1,2,3]);

    // 条件删除
    // 删除状态为0的数据
    User::destroy(['status' => 0]);

    // 使用闭包删除
    User::destroy(function($query){
        $query->where('id','>',10);
    });

    // 通过数据库类的查询条件删除
    User::where('id','>',10)->delete();
```

### 查询

#### 查询单个数据

```php
    // 取出主键为1的数据
    $user = User::get(1);
    echo $user->name;

    // 使用数组查询
    $user = User::get(['name' => 'thinkphp']);

    // 使用闭包查询
    $user = User::get(function($query){
        $query->where('name', 'thinkphp');
    });
    echo $user->name;
```

```php
    $user = new User();
    // 查询单个数据
    $user->where('name', 'thinkphp')
         ->find();
```


#### 获取多个数据

```php
    // 根据主键获取多个数据
    $list = User::all('1,2,3');
    // 或者使用数组
    $list = User::all([1,2,3]);
    foreach($list as $key=>$user){
        echo $user->name;
    }
    // 使用数组查询
    $list = User::all(['status'=>1]);
    // 使用闭包查询
    $list = User::all(function($query){
        $query->where('status', 1)->limit(3)->order('id', 'asc');
    });
    foreach($list as $key=>$user){
        echo $user->name;
    }
```

#### 动态查询

```php
    // 根据name字段查询用户
    $user = User::getByName('thinkphp');

    // 根据email字段查询用户
    $user = User::getByEmail('thinkphp@qq.com');
```

#### 查询缓存

`get`方法和`all`方法的第三个参数表示是否使用查询缓存，或者设置缓存标识。

```php
    $user = User::get(1,'',true);
    $list  = User::all('1,2,3','',true);
```

## 修改器

修改器的作用是可以在数据赋值的时候自动进行转换处理，例如：

```php
    class User extends Model 
    {
        public function setNameAttr($value)
        {
            return strtolower($value);
        }
    }
```

也可以进行序列化字段的组装：

```php
    class User extends Model 
    {
        public function setNameAttr($value,$data)
        {
            return serialize($data);
        }
    }
```

## 时间戳

第一种方式，是在数据库配置文件中添加全局设置：

```php
    // 开启自动写入时间戳字段
    'auto_timestamp' => true,
```

第二种是直接在单独的模型类里面设置：

```php
    protected $autoWriteTimestamp = true;
```

如果这两个地方设置为`true`，默认识别为整型`int`类型，如果你的时间字段不是int类型的话，例如使用`datetime`类型的话，可以这样设置：

```php
    // 开启自动写入时间戳字段
    'auto_timestamp' => 'datetime',

    // 或者
    protected $autoWriteTimestamp = 'datetime';
```


## 软删除

```php
    namespace app\index\model;

    use think\Model;
    use traits\model\SoftDelete;

    class User extends Model
    {
        use SoftDelete;
        protected $deleteTime = 'delete_time';
    }
```


```php
    // 软删除
    User::destroy(1);
    // 真实删除
    User::destroy(1,true);
    $user = User::get(1);
    // 软删除
    $user->delete();
    // 真实删除
    $user->delete(true);
```

默认情况下查询的数据不包含软删除数据，如果需要包含软删除的数据，可以使用下面的方式查询：

```php
    User::withTrashed()->find();
    User::withTrashed()->select();

    // 只看软删除
    User::onlyTrashed()->find();
    User::onlyTrashed()->select();
```

## JSON序列化

```php
    $user = User::get(1);
    echo $user->toJson();
```

可以设置无需输出的字段，例如：

```php
    $user = User::get(1);
    echo $user->hidden(['create_time','update_time'])->toJson();
```

或者追加其它的字段：

```php
    $user = User::get(1);
    echo $user->append(['status_text'])->toJson();
```

设置允许输出的属性：

```php
    $user = User::get(1);
    echo $user->visible(['id','name','email'])->toJson();
```

模型对象可以直接被JSON序列化，例如：
```php
    echo json_encode(User::get(1));
```

### 追加关联模型的属性

```php
    $user = User::find(1);
    echo $user->appendRelationAttr('profile',['email','nickname'])->toJson();
```


## 关联

### 一对一关联

定义一对一关联，例如，一个用户都有一个个人资料，我们定义`User`模型如下：

```php
    namespace app\index\model;

    use think\Model;

    class User extends Model
    {
        public function profile()
        {
            return $this->hasOne('Profile');
        }
    }
```

`hasOne`方法的参数包括：

> hasOne('关联模型名','外键名','主键名',['模型别名定义'],'join类型');

#### 定义相对的关联

我们可以在`Profile`模型中定义一个相对的关联关系，例如：

```php
    namespace app\index\model;

    use think\Model;

    class Profile extends Model 
    {
        public function user()
        {
            return $this->belongsTo('User');
        }
    }
```


`belongsTo`的参数包括：

> belongsTo('关联模型名','外键名','关联表主键名',['模型别名定义'],'join类型');

### 一对多关联

使用`hasMany`方法定义:

> hasMany('关联模型名','外键名','主键名',['模型别名定义']);

```php
    <?php

    namespace app\index\model;

    use think\Model;

    class Article extends Model 
    {
        public function comments()
        {
            return $this->hasMany('Comment');
        }
    }
```

#### 定义相对的关联

```php
    <?php

    name app\index\model;

    use think\Model;

    class Comment extends Model 
    {
        public function article()
        {
            return $this->belongsTo('article');
        }
    }
```


### 多对多关联

例如，我们的用户和角色就是一种多对多的关系，我们在User模型定义如下

```php
    <?php

    namespace app\index\model;

    use think\Model;

    class User extends Model 
    {
        public function roles()
        {
            return $this->belongsToMany('Role');
        }
    }
```

`belongsToMany`方法的参数如下：

> belongsToMany('关联模型名','中间表名','外键名','当前模型关联键名',['模型别名定义']);

#### 定义相对的关联

我们可以在`Role`模型中定义一个相对的关联关系，例如：

```php
    <?php

    namespace app\index\model;

    use think\Model;

    class Role extends Model 
    {
        public function users()
        {
            return $this->belongsToMany('User');
        }
    }
```


### 多态关联

```php
    <?php

    namespace app\index\model;

    use think\Model;

    class Article extends Model
    {
        /**
         * 获取所有针对文章的评论。
         */
        public function comments()
        {
            return $this->morphMany('Comment', 'commentable');
        }
    }
```

`morphMany`方法的参数如下：

> morphMany('关联模型名','多态字段信息','多态类型');


### 延迟预载入

如果你的数据集查询返回的是数据集对象，可以使用调用数据集对象的`load`实现延迟预载入：

```php
    // 查询数据集
    $list = User::all([1,2,3]);

    // 延迟预载入
    $list->load('cards');
    foreach($list as $user){
        // 获取用户关联的card模型数据
        dump($user->cards);
    }
```

如果你的数据集查询返回的是数组，系统提供了一个`load_relation`助手函数可以完成同样的功能。

```php
    // 查询数据集
    $list = User::all([1,2,3]);

    // 延迟预载入
    $list = load_relation($list,'cards');
    foreach($list as $user){
        // 获取用户关联的card模型数据
        dump($user->cards);
    }
```


# 视图

## 视图实例化

```php
     // 渲染模板输出
    return $this->fetch('hello',['name'=>'thinkphp']);
```

### 助手函数

```php
    return view('hello',['name'=>'thinkphp']);
```

## 模板赋值

### `assign`方法

```php
    namespace index\app\controller;

    class Index extends \think\Controller
    {
        public function index()
        {
            // 模板变量赋值
            $this->assign('name','ThinkPHP');
            $this->assign('email','thinkphp@qq.com');
            // 或者批量赋值
            $this->assign([
                'name'  => 'ThinkPHP',
                'email' => 'thinkphp@qq.com'
            ]);
            // 模板输出
            return $this->fetch('index');
        }
    }
```

### 助手函数

```php
    return view('index', [
        'name'  => 'ThinkPHP',
        'email' => 'thinkphp@qq.com'
    ]);
```

### `share`方法 

```php
    think\View::share('name','value');
    // 或者批量赋值
    think\View::share(['name1'=>'value','name2'=>'value2']);
```

## 模板渲染

```php
    // 不带任何参数 自动定位当前操作的模板文件
    return $this->fetch();

    // 指定模板输出
    return $this->fetch('edit'); 

    // 渲染完整模板
    return $this->fetch('./template/public/menu.html');

    // 渲染内容
    return $this->display($content,$vars);
```


# 日志

## 日志写入

由于系统在请求结束后会自动调用`Log::save`方法，所以通常，你只需要调用`Log::record`记录日志信息即可。

|方法|描述|
|:----|:----|
|`Log::record()`|记录日志信息到内存|
|`Log::save()`|把保存在内存中的日志信息|
|`Log::write`|实时写入一条日志信息|

```php
    Log::record('测试日志信息');
```

默认的话记录的日志级别是INFO，也可以指定日志级别：

```php
    Log::record('测试日志信息，这是警告级别','notice');
```

采用`record`方法记录的日志信息不是实时保存的，如果需要实时记录的话，可以采用`write`方法，例如

```php
    Log::write('测试日志信息，这是警告级别，并且实时写入','notice');
```


### 日志级别

- `log` 常规日志，用于记录日志
- `error` 错误，一般会导致程序的终止
- `notice` 警告，程序可以运行但是还不够完美的错误
- `info` 信息，程序输出信息
- `debug` 调试，用于调试信息
- `sql` SQL语句，用于SQL记录，只在数据库的调试模式开启时有效

### 助手函数

```php
    trace('错误信息','error');
    trace('日志信息','info');
```

支持指定级别日志的输入，需要配置信息：

```php
    'log'   => [
        'type'  => 'File',
        // 日志记录级别，使用数组表示
        'level' => ['error'],
    ],
```

### 单文件日志

```php
    'log'   => [
        'type'  => 'File',
        // 日志记录级别，使用数组表示
        'single' => true,
    ],
```


## 独立日志

为了便于分析，`File`类型的日志驱动还支持设置某些级别的日志信息单独文件记录，例如：

```php
    'log'   => [
        'type'          => 'file', 
        // error和sql日志单独记录
        'apart_level'   =>  ['error','sql'],
    ],
```

## 日志清空

```php
    Log::clear();
```

## 写入授权

首先需要在应用配置文件或者应用公共文件中添加当前访问的授权Key定义，例如：

```php
    // 设置IP为授权Key
    Log::key(Request::instance()->ip());
```

然后在日志配置参数中增加`allow_key`参数，如下：

```php
    'log'   =>  [
        // 日志类型为File
        'type'      =>  'File',
        // 授权只有202.12.36.89 才能记录日志
        'allow_key' =>  ['202.12.36.89'],
    ]
```



# 错误和调试

## 调试模式

`.env`文件的定义格式如下：

```php
    // 设置开启调试模式
    app_debug =  true
    // 其它的环境变量设置
    // ...
```

> 定义了`.env`文件后，配置文件中定义`app_debug`参数无效。

一旦关闭调试模式，发生错误后不会提示具体的错误信息，如果你仍然希望看到具体的错误信息，那么可以如下设置：

```php
    // 显示错误信息
    'show_error_msg'        =>  true,
```

## 异常处理

```php
    // 异常错误报错级别,
    error_reporting(E_ERROR | E_PARSE );
```

### 部署模式异常

```php
    // 显示错误信息
    'show_error_msg'        =>  true,    
```

## 异常捕获

```php
    try{
        Db::name('user')->find();
        $this->success('执行成功!');
    }catch(\Exception $e){
        $this->error('执行错误');
    }
```

应该改成:

```php
    try{
        Db::name('user')->find();
    } catch (\Exception $e) {
        $this->error('执行错误');
    }

    $this->success('执行成功!');
```


## 抛出异常

### 手动抛出异常

可以使用 `\think\Exception` 类来抛出异常

```php
    // 使用think自带异常类抛出异常
    throw new \think\Exception('异常消息', 100006);
```

如果不使用think异常类，也可以定义自己的异常类来抛出异常

```php
    throw new \foobar\Exception('异常消息');
```

也可以使用系统提供的助手函数来简化处理：

```php
    exception('异常消息', 100006);

    // 使用自定义异常类
    exception('异常消息', 100006, \foobar\Exceeption);
```

### 抛出 HTTP 异常

```php
    // 抛出 HTTP 异常
    throw new \think\exception\HttpException(404, '异常消息', null, [参数]);
```

#### 助手函数

```php
    abort(404, '异常消息', [参数])
```


## `Trace` 调试

`Trace`调试功能就是ThinkPHP提供给开发人员的一个用于开发调试的辅助工具。可以实时显示当前页面的操作的请求信息、运行情况、SQL执行、错误提示等，并支持自定义显示，5.0版本的Trace调试支持没有页面输出的操作调试。

### 开启 `Trace` 调试

```php
    // 开启应用Trace调试
    'app_trace' =>  true,
```

### 页面`Trace`显示

```php
    // Trace信息
    'trace'     =>  [
        //支持Html,Console
        'type'  =>  'html',
    ]
```

### 浏览器`Trace`显示

```php
    // Trace信息
    'trace' =>[
        // 使用浏览器console输出trace信息
        'type'  =>  'console',
    ] 
```


## 性能调试

```php
    Debug::remark('begin');
    // ...其他代码段
    Debug::remark('end');
    // ...也许这里还有其他代码
    // 进行统计区间
    echo Debug::getRangeTime('begin','end').'s';
```

### 助手函数

```php
    debug('begin');
    // ...其他代码段
    debug('end');
    // ...也许这里还有其他代码
    // 进行统计区间
    echo debug('begin','end').'s';
    echo debug('begin','end',6).'s';
    echo debug('begin','end','m').'kb';
```


## SQL调试

### 监听SQL

```php
    Db::listen(function($sql,$time,$explain){
        // 记录SQL
        echo $sql. ' ['.$time.'s]';
        // 查看性能分析结果
        dump($explain);
    });
```

### 调试执行的SQL语句

```php
    User::get(1);
    echo User::getLastSql();
```

也可以使用fetchSql方法直接返回当前的查询SQL而不执行，例如：

```php
    echo User::fetchSql()->find(1);
```


## 远程调试

### `Socket`调试

```php
    'log' =>  [
        'type'                => 'socket',
        'host'                => 'slog.thinkphp.cn',
        //日志强制记录到配置的client_id
        'force_client_ids'    => [],
        //限制允许读取日志的client_id
        'allow_client_ids'    => [],
    ]
```

## `404`页面

一旦抛出了`HttpException`异常，可以支持定义单独的异常页面的模板地址，只需要在应用配置文件中增加：

```php
    'http_exception_template'    =>  [
        // 定义404错误的重定向页面地址
        404 =>  APP_PATH.'404.html',
        // 还可以定义其它的HTTP status
        401 =>  APP_PATH.'401.html',
    ]
```

一般来说HTTP异常是由系统自动抛出的，但我们也可以手动抛出

```php
    throw new \think\exception\HttpException(404, '页面不存在');
```

### 助手函数

```php
    abort(404,'页面不存在');
```


# 验证

## 验证器

### 独立验证

```php
    $validate = new Validate([
        'name'  => 'require|max:25',
        'email' => 'email'
    ]);
    $data = [
        'name'  => 'thinkphp',
        'email' => 'thinkphp@qq.com'
    ];
    if (!$validate->check($data)) {
        dump($validate->getError());
    }
```

### 验证器

```php
    namespace app\index\validate;

    use think\Validate;

    class User extends Validate
    {
        protected $rule = [
            'name'  =>  'require|max:25',
            'email' =>  'email',
        ];

    }
```

```php
    $data = [
        'name'=>'thinkphp',
        'email'=>'thinkphp@qq.com'
    ];

    $validate = Loader::validate('User');

    if(!$validate->check($data)){
        dump($validate->getError());
    }
```

使用助手函数实例化验证器

```php
    $validate = validate('User');
```

## 验证规则

```php
    $rule = [
        'name'  => 'require|max:25',
        'age'   => 'number|between:1,120',
        'email' => 'email',
    ];

    $msg = [
        'name.require' => '名称必须',
        'name.max'     => '名称最多不能超过25个字符',
        'age.number'   => '年龄必须是数字',
        'age.between'  => '年龄只能在1-120之间',
        'email'        => '邮箱格式错误',
    ];

    $data = [
        'name'  => 'thinkphp',
        'age'   => 10,
        'email' => 'thinkphp@qq.com',
    ];

    $validate = new Validate($rule, $msg);
    $result   = $validate->check($data);
```

### 自定义验证规则

```php
    // 自定义验证规则
    protected function checkName($value,$rule,$data)
    {
        return $rule == $value ? true : '名称错误';
    }
```

## 控制器验证

```php
    $result = $this->validate(
        [
            'name'  => 'thinkphp',
            'email' => 'thinkphp@qq.com',
        ],
        [
            'name'  => 'require|max:25',
            'email'   => 'email',
        ]);
    if(true !== $result){
        // 验证失败 输出错误信息
        dump($result);
    }
```

如果定义了验证器类的话，例如：

```php
    namespace app\index\validate;

    use think\Validate;

    class User extends Validate
    {
        protected $rule = [
            'name'  =>  'require|max:25',
            'email' =>  'email',
        ];
        
        protected $message = [
            'name.require'  =>  '用户名必须',
            'email' =>  '邮箱格式错误',
        ];
        
        protected $scene = [
            'add'   =>  ['name','email'],
            'edit'  =>  ['email'],
        ];
    }
```

控制器中的验证代码可以简化为：

```php
    $result = $this->validate($data,'User');
    if(true !== $result){
        // 验证失败 输出错误信息
        dump($result);
    }
```

## 模型验证

```php
    $User = new User;
    $result = $User->validate(
        [
            'name'  => 'require|max:25',
            'email'   => 'email',
        ],
        [
            'name.require' => '名称必须',
            'name.max'     => '名称最多不能超过25个字符',
            'email'        => '邮箱格式错误',
        ]
    )->save($data);
    if(false === $result){
        // 验证失败 输出错误信息
        dump($User->getError());
    }
```

## 内置规则

> https://www.kancloud.cn/manual/thinkphp5/129356


## 静态调用

```php
    // 日期格式验证
    Validate::dateFormat('2016-03-09','Y-m-d'); // true
    // 验证是否有效的日期
    Validate::is('2016-06-03','date'); // true
    // 验证是否有效邮箱地址
    Validate::is('thinkphp@qq.com','email'); // true
    // 验证是否在某个范围
    Validate::in('a',['a','b','c']); // true
    // 验证是否大于某个值
    Validate::gt(10,8); // true
    // 正则验证
    Validate::regex(100,'\d+'); // true
```



# 杂项

## Session

### 赋值

```php
    // 赋值（当前作用域）
    Session::set('name','thinkphp');
    // 赋值think作用域
    Session::set('name','thinkphp','think');
```


### 判断是否存在

```php
    // 判断（当前作用域）是否赋值
    Session::has('name');
    // 判断think作用域下面是否赋值
    Session::has('name','think');
```


### 取值

```php
    // 取值（当前作用域）
    Session::get('name');
    // 取值think作用域
    Session::get('name','think');
```


### 删除

```php
    // 删除（当前作用域）
    Session::delete('name');
    // 删除think作用域下面的值
    Session::delete('name','think');
```


### 指定作用域

```php
    // 指定当前作用域
    Session::prefix('think');
```

### 取值并删除

```php
    // 取值并删除
    Session::pull('name');
```

### 清空

```php
    // 清除session（当前作用域）
    Session::clear();
    // 清除think作用域
    Session::clear('think');
```


### 助手函数

```php
    // 初始化session
    session([
        'prefix'     => 'module',
        'type'       => '',
        'auto_start' => true,
    ]);

    // 赋值（当前作用域）
    session('name', 'thinkphp');

    // 赋值think作用域
    session('name', 'thinkphp', 'think');

    // 判断（当前作用域）是否赋值
    session('?name');

    // 取值（当前作用域）
    session('name');

    // 取值think作用域
    session('name', '', 'think');

    // 删除（当前作用域）
    session('name', null);

    // 清除session（当前作用域）
    session(null);

    // 清除think作用域
    session(null, 'think');
```


## Cookie

### 初始化

```php
    // cookie初始化
    Cookie::init(['prefix'=>'think_','expire'=>3600,'path'=>'/']);
    // 指定当前前缀
    Cookie::prefix('think_');
```

### 设置

```php
    // 设置Cookie 有效期为 3600秒
    Cookie::set('name','value',3600);
    // 设置cookie 前缀为think_
    Cookie::set('name','value',['prefix'=>'think_','expire'=>3600]);
    // 支持数组
    Cookie::set('name',[1,2,3]);
```


### 获取

```php
    Cookie::get('name');
    // 获取指定前缀的cookie值
    Cookie::get('name','think_');
```


### 助手函数

```php
    // 初始化
    cookie(['prefix' => 'think_', 'expire' => 3600]);

    // 设置
    cookie('name', 'value', 3600);

    // 获取
    echo cookie('name');

    // 删除
    cookie('name', null);

    // 清除
    cookie(null, 'think_');
```

## 上传

假设表单代码如下：

```php
    <form action="/index/index/upload" enctype="multipart/form-data" method="post">
        <input type="file" name="image" /> <br> 
        <input type="submit" value="上传" /> 
    </form>  
```

```php
    public function upload()
    {
        // 获取表单上传文件 例如上传了001.jpg
        $file = request()->file('image');
        
        // 移动到框架应用根目录/public/uploads/ 目录下
        if($file){
            $info = $file->move(ROOT_PATH . 'public' . DS . 'uploads');
            if($info){
                // 成功上传后 获取上传信息
                // 输出 jpg
                echo $info->getExtension();
                // 输出 20160820/42a79759f284b767dfcb2a0197904287.jpg
                echo $info->getSaveName();
                // 输出 42a79759f284b767dfcb2a0197904287.jpg
                echo $info->getFilename(); 
            }else{
                // 上传失败获取错误信息
                echo $file->getError();
            }
        }
    }
```


### 多文件上传

```php
    <form action="/index/index/upload" enctype="multipart/form-data" method="post">
        <input type="file" name="image[]" /> <br> 
        <input type="file" name="image[]" /> <br> 
        <input type="file" name="image[]" /> <br> 
        <input type="submit" value="上传" /> 
    </form> 
```

控制器代码可以改成：

```php
    public function upload()
    {
        // 获取表单上传文件
        $files = request()->file('image');
        foreach($files as $file){
            // 移动到框架应用根目录/public/uploads/ 目录下
            $info = $file->move(ROOT_PATH . 'public' . DS . 'uploads');
            if($info){
                // 成功上传后 获取上传信息
                // 输出 jpg
                echo $info->getExtension(); 
                // 输出 42a79759f284b767dfcb2a0197904287.jpg
                echo $info->getFilename(); 
            }else{
                // 上传失败获取错误信息
                echo $file->getError();
            }    
        }
    }
```


## 验证码

首先使用Composer安装`think-captcha`扩展包：

```php
    composer require topthink/think-captcha;
```

使用TP5的内置验证功能，添加captcha验证规则即可:

```php
    $this->validate($data,[
        'captcha|验证码'=>'require|captcha'
    ]);
```

或者手动验证

```php
    if(!captcha_check($captcha)){
     //验证失败
    };
```

### 验证码的自定义用法

如果项目未开启路由，或者有实际需求可自行调用Captcha类操作

验证码的生成：

```php
    $captcha = new Captcha();
    return $captcha->entry();
```

验证码刷新 写个`onclick="this.src=this.src+'?'"` 就行


# 扩展

## 函数

扩展系统函数

```php
    // 增加一个新的table助手函数
    function table($table, $config = [])
    {
        return \think\Db::connect($config)->setTable($table);
    }

    // 替换已有的db助手函数
    function db($name, $config= [])
    {
        return \think\Db::connect($config)->name($name); 
    }
```


## 驱动

以缓存驱动为例，如果我们扩展了一个自己的`redis`驱动，类名为
`app\driver\cache\Redis`，那么我们只需要设置缓存类型为：


```php
    'cache' => [
        // 驱动方式
        'type'   => '\app\driver\cache\Redis',
        // 缓存前缀
        'prefix' => '',
        // 缓存有效期 0表示永久缓存
        'expire' => 0,
    ]
```

# 命令行

## 自动生成目录结构

首先需要定义一个用于自动生成的规则定义文件，通常命名为 `build.php`

```php
    return [
        // 生成运行时目录
        '__file__' => ['common.php'],

        // 定义index模块的自动生成
        'index'    => [
            '__file__'   => ['common.php'],
            '__dir__'    => ['behavior', 'controller', 'model', 'view'],
            'controller' => ['Index', 'Test', 'UserType'],
            'model'      => [],
            'view'       => ['index/index'],
        ],
        // 。。。 其他更多的模块定义
    ];
```

- `__dir__` 表示生成目录（支持多级目录）
- `__file__` 表示生成文件（不定义默认会生成 `config.php` 文件）
- `controller` 表示生成`controller`类
- `model`表示生成`model`类
- `view`表示生成`html`文件（支持子目录）

## 快速创建类库文件

### 快速生成控制器类

```php
    php think make:controller index/Blog
```

### 快速生成模型类

```php
    php think make:model index/Blog
```

## 优化

- 生成类库映射文件`optimize:autoload`

```php
    php think optimize:autoload
```

- 生成路由缓存`optimize:route`

```php
    php think optimize:route
```

- 清除缓存文件`clear`

```php
    php think clear
```

- 生成配置缓存`optimize:config`

```php
    php think optimize:config
```

- 生成数据表字段缓存`optimize:schema`

```php
    php think optimize:schema
```


# 附录

## 配置参考

### 预定义常量

```php
    EXT           类库文件后缀（.php）
    THINK_VERSION 框架版本号  
```

### 路径常量

```php
    DS 当前系统的目录分隔符
    THINK_PATH 框架系统目录 
    ROOT_PATH 框架应用根目录
    APP_PATH 应用目录（默认为application）
    CONF_PATH 配置目录（默认为APP_PATH）
    LIB_PATH 系统类库目录（默认为 THINK_PATH.'library/'）
    CORE_PATH 系统核心类库目录 （默认为 LIB_PATH.'think/'）
    TRAIT_PATH 系统trait目录（默认为 LIB_PATH.'traits/'）
    EXTEND_PATH 扩展类库目录（默认为 ROOT_PATH . 'extend/')
    VENDOR_PATH 第三方类库目录（默认为 ROOT_PATH . 'vendor/'）
    RUNTIME_PATH 应用运行时目录（默认为 ROOT_PATH.'runtime/'）
    LOG_PATH 应用日志目录 （默认为 RUNTIME_PATH.'log/'）
    CACHE_PATH 项目模板缓存目录（默认为 RUNTIME_PATH.'cache/'）
    TEMP_PATH 应用缓存目录（默认为 RUNTIME_PATH.'temp/'）
``` 

### 系统常量

```php
    IS_WIN 是否属于Windows 环境  
    IS_CLI 是否属于命令行模式  
    THINK_START_TIME 开始运行时间（时间戳）
    THINK_START_MEM 开始运行时候的内存占用
    ENV_PREFIX 环境变量配置前缀
```


# 参考

- [ThinkPHP5.0完全开发手册](https://www.kancloud.cn/manual/thinkphp5/118003)





