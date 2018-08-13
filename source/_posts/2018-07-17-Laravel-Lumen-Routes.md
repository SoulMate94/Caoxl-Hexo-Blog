---
title: Laravel/Lumen/Dingo 的路由
date: 2018-07-17 09:17:12
categories: Laravel
tags: [Laravel, Lumen, Dingo, Framework]
---

> Laravel/Lumen/Dingo 的路由区别

<!-- more -->

# Laravel

## 基础路由

```
    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);
```

> 有的时候你可能需要注册一个可响应多个 `HTTP` 请求的路由，这时你可以使用 `match` 方法，也可以使用 `any` 方法注册一个实现响应所有 HTTP 请求的路由：

```
    Route::match(['get', 'post'], '/', function () {
        // 
    });
    
    Route::any('foo', function () {
        //
    });
```

## 路由组

### 中间件

```
    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // 使用 first 和 second 中间件
        });
    });
```

### 命名空间

```
    Route::namespace('Admin')->group(function () {
        // 在 "App\Http\Controllers\Admin" 命名空间下的控制器
    });
```

### 路由前缀

```
    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // 匹配包含 "/admin/users" 的 URL
        };
    });
```

## 路由组-`group`

```
    Route::group([
        'namespace'  => 'Admin',
        'prefix'     => 'admin',
        'middleware' => 'auth',
        'domain'     => 'blog.domain.com'
    ], function () {
        Route::get('test', function () {
            //
        });
    });
```

# Lumen

## `bootstrap/app.php`

```php
    <?php
    
    $app->group(['namespace' => 'App\Http\Controllers'], function ($app) {
        require_once __DIR__.'/../routes/web.php';
        require_once __DIR__.'/../routes/api.php';
    });
    
    return $app;
```

> 上面括号中使用的是`$app` 所以在路由文件中也使用`$app`

- `route.php`

```php
    <?php
    
    $app->group([
        'namespace'  => 'Test',
        'prefix'     => 'test',
        'middleware' => 'auth',
        'domain'     => 'test.domain.com'
    ], function ($app) {
        $app->get('/', 'Index@index');
        $app->post('login', 'AuthController@login');
    });
```

```
    $app->get('/', function () use ($app) {
        return response()->json([
            'err' => '403,
            'msg' => 'Forbidden',
        ], 403);
    });
```

# Dingo

## 原生

- `route.php`

```php
    <?php
    $api = app('Dingo\Api\Routing\Router');
    
    $api->version('v1', [
        'namespace' => 'App\Api\V1',
    ], function ($router) {
        $router->group([
            'prefix'     => 'test',
            'middleware' => [
                'api.auth',
            ],
        ], function ($router) {
            $router->get('/', 'Test@index');
            $router->post('wechat', 'Passport@login')->name('wechat.login');
        });
    });
```

## 封装后

- `functions.php`

```php
    <?php

    if (! function_exists('fe')) {
        function fe(string $name) {
            return function_exists($name);
        }
    }    
    
    if (! fe('dingo')) {
        function dingo() {
            return app('Dingo\Api\Routing\Router');
        }
    }
```

- `route.php`

```php
    <?php
    
    dingo()->version('v1', [
        'namespace' => 'App\Api\V1',
    ], function ($router) {
        $router->group([
            'prefix'     => 'test',
            'middleware' => [
                'api.auth',
            ],
        ], function ($router) {
            $router->get('/', 'Test@index');
            $router->post('wechat', 'Passport@login')->name('wechat.login');
        });
    });
```

