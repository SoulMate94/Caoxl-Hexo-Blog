---
title: 基于Laravel MongoDB的部分实践
date: 2018-08-16 11:26:02
categories: Laravel
tags: [Laravel, MongoDB]
---

> 原文请看我的好朋友葉蕓榕的博客: [laravel与MongoDB 的部分实践](http://yeyunrong.com/2018/04/02/laravel%E4%B8%8EMongoDB-%E7%9A%84%E9%83%A8%E5%88%86%E5%AE%9E%E8%B7%B5/)

<!-- more -->

# MongoDB客户端安装

> [Windows 平台安装 MongoDB](http://www.runoob.com/mongodb/mongodb-window-install.html)


```
    PS C:\Program Files\MongoDB\Server\4.0\bin> ./mongo
    MongoDB shell version v4.0.1
    connecting to: mongodb://127.0.0.1:27017
    MongoDB server version: 4.0.1
    Welcome to the MongoDB shell.
    For interactive help, type "help".
    For more comprehensive documentation, see
            http://docs.mongodb.org/
    Questions? Try the support group
            http://groups.google.com/group/mongodb-user
    Server has startup warnings:
    2018-08-16T14:36:30.219+0800 I CONTROL  [initandlisten]
    2018-08-16T14:36:30.219+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
    2018-08-16T14:36:30.220+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
    2018-08-16T14:36:30.220+0800 I CONTROL  [initandlisten]
    ---
    Enable MongoDB's free cloud-based monitoring service, which will then receive and display
    metrics about your deployment (disk utilization, CPU, operation statistics, etc).
    
    The monitoring data will be available on a MongoDB website with a unique URL accessible to you
    and anyone you share the URL with. MongoDB may use this information to make product
    improvements and to suggest MongoDB products and deployment options to you.
    
    To enable free monitoring, run the following command: db.enableFreeMonitoring()
    To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
```

# MongoDB扩展安装

这里做下补充, 在windows环境下开发需要安装`mongodb`的扩展

- 首先, `php -m` 查看是否已有`mongodb`的扩展

- 其次, `php -v` 查看环境的版本

```
    $ php -v
    PHP 7.2.1 (cli) (built: Jan  4 2018 04:28:54) ( NTS MSVC15 (Visual C++ 2017) x86
     )
    Copyright (c) 1997-2017 The PHP Group
    Zend Engine v3.2.0, Copyright (c) 1998-2017 Zend Technologies
```

- 下载对应版本的`mongodb`扩展

> https://pecl.php.net/package/mongodb/1.5.2/windows

# `Laravel`链接`MongoDB`

- 先在项目目录下运行如下代码

```
    composer require jenssegers/mongodb
```

- 在配置信息中添加如下信息:

  - `config.php/app.php` 下的 `providers` 添加:
  ```
        Jenssegers\Mongodb\MongodbServiceProvider::class,
  ```
  - `config.php/app.php` 下的 `aliases` 添加:
  ```
        'Mongo' => Jenssegers\Mongodb\MongodbServiceProvider::class,
        'MoEloquent' => Jenssegers\Mongodb\Eloquent\Model::class,
  ```
  - `config.php/database.php` 中的 `connections` 添加:
  ```
        'mongodb' => [
            'driver'   => 'mongodb',
            'host'     => env('MONGO_HOST', 'localhost'),
            'port'     => env('MONGO_PORT', '27017'),
            'database' => env('MONGO_DATABASE', 'test_mongo'),
            'username' => env('MONGO_USERNAME', 'root'),
            'password' => env('MONGO_PASSWORD', 'root'),
            'strict'   => false,
        ],
  ```
  - 在 `.env`中添加:
  ```
        MONGO_HOST=127.0.0.1
        MONGO_PORT=27017
        MONGO_database=test_mongo
        MONGO_USERNAME=
        MONGO_PASSWORD=
  ```


# 添加测试路由

- 在 `routes/web.php` 中添加:

```
    Route::get('/test_mongo','TestMongoDBController@testMongo');
```

# 添加测试Model

- `app\Models\MongoDB.php`

```
    <?php
    
    namespace App\Models;
    
    # 已经在aliases添加即可不用下面这么
    # use Jenssegers\Mongodb\Eloquent\Model as MoEloquent;
    use Illuminate\Support\Facades\DB;
    
    class MongoDB extends MoEloquent
    {
        protected $connection = "mongodb";
        protected $collection = "test_mongo";
    }
```

# 添加测试控制器

- `app\Http\Controllers\TestMongoDBController`

```
    <?php
    
    namespace App\Http\Controllers;
    
    use App\Models\MongoDB;
    
    class TestMongoDBController extends Controller
    {
        public function testMongo()
        {
            dd($this->testUpdate());
        }
    
        public function testAdd()
        {
            $mongo = new MongoDB();
    
            $mongo->name = 'test_name_1';
            $mongo->info = 'test_info_1';
    
            $res = $mongo->save();
    
            return $res ? '插入成功' : '插入失败';
        }
    
        public function testDel()
        {
            // 删除指定id
            $res = MongoDB::destroy('5b75205aa8ce413058001c42');
    
            return $res ? '删除成功' : '删除失败';
        }
    
        public function testUpdate()
        {
            $info = MongoDB::where('_id', '=', '5b752088a8ce413058001c43')->first();
            $info->info = '更新后的信息';
            $res = $info->save();
    
            return $res ? '更新成功' : '更新失败';
        }
    }
```

如下图:

![MongoDB](https://caoxl.com/images/laravel-mongo.png)

# 参考

- [Laravel MongoDB--jenssegers/laravel-mongodb](https://github.com/jenssegers/laravel-mongodb)