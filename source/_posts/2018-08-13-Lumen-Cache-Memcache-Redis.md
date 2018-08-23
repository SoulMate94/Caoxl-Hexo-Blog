---
title: 基于Lumen-几种缓存方法
date: 2018-08-13 10:59:07
categories: Lumen
tags: [Lumen, Memcache, Redis]
---

> 缓存时PHPer优化所必备的一个知识点

<!-- more -->

# 配置

首先，修改`.env`文件配置缓存的驱动方式

```
    // 缓存驱动方式：apc, array, database, file, memcached, redis
    CACHE_DRIVER=memcached
```

- `apc`: APC驱动，APC是PHP的一个扩展，其目标是为缓存和优化PHP中间码（opcode）提供一个免费、开源、健壮的框架
- `array`: 数组驱动，往往仅仅用于测试，好处是不会持久化，只会在一次PHP脚本执行的生命周期内有效
- `database`: 数据库驱动，需参考下面代码在数据库中建立缓存表，并设置`CACHE_DATABASE_TABLE`和`CACHE_DATABASE_CONNECTION`

```
    Schema::create('cache', function($table) {
      $table->string('key')->unique();
      $table->text('value');
      $table->integer('expiration');
    });    
```

- `file`: 文件驱动，系统将生成缓存文件储存在`/storage/framework/cache`文件夹里，所以，**777**权限是必须的
- `memcached`: `Memcached`驱动，使用之前需要先在系统中安装`Memcached`，并设置`MEMCACHED_HOST和MEMCACHED_PORT`
- `redis`: `Redis`驱动，使用之前需要先在系统中安装`Redis`，并设置`CACHE_REDIS_CONNECTION`
           
> 众多驱动方式中，`array`不是持久缓存，`database`需要读取数据库，`file`需要读取文件，`apc/memcached/redis`是基于内存的，所以...选择适合自己的

# 使用

> 缓存需要`Facade`的支持，否则会提示`Class 'Cache' not found`错误，所以，请检查`/bootstrap/app.php`，确认`$app->withFacades();`的注释已去掉
> > Lumen已经默认去掉注释

无论选用任何驱动方式，缓存的使用方式是统一的，请看以下代码：

```
    use Cache;
    ...
    
    $id = 5;
    // 读取缓存
    $result = Cache::get('UserData_'.$id);
    if($result){
        return $result;
    }
    
    $result = User::findOrFail($id);
    // 添加缓存，时间60分钟
    Cache::put('UserData_'.$id, $result, 60);
```

接下来具体说一下`Memcached`和`Redis`

# `Memcached`

## `Memcache` 与 `Memcached`

> `Memcache:` 是一个自由和开放源代码、高性能、分配的内存对象缓存系统。用于加速动态web应用程序，减轻数据库负载。它可以应对任意多个连接，使用非阻塞的网络IO。由于它的工作机制是在内存中开辟一块空间，然后建立一个Hash表，Memcached自管理这些Hash表。
> 
> `Memcached:` Memcache是该系统的项目名称，Memcached是该系统的主程序文件（字母d可以理解为daemon），以守护程序方式运行于一个或多个服务器中，随时接受客户端的连接操作，使用共享内存存取数据。

PHP 有两个 Memcached 客户端：**“PHP Memcache 扩展”** 和 **“PHP Memcached 扩展”**，这就是是我们搞混的地方。

- **PHP Memcache 扩展**: 用 PHP 实现的，支持面向对象和面向过程两种接口，2004年就实现了，是老客户端，而且功能少，属性也可设置的少。
> 函数列表：http://php.net/manual/zh/book.memcache.php

- **PHP Memcached 扩展**: 基于 libmemcached 开发的，使用 libmemcached 库提供的 API 与 Memcached 服务进行交互，只支持面向对象的接口，2009年才实现，Memcached 扩展功能更加完善，支持的函数更多，比如支持批量操作，
> 函数列表：http://php.net/manual/zh/book.memcached.php
**现在一般建议使用 Memcached 扩展**。

### 必须同时安装服务端和客户端

如果安装了 `Memcached` 服务端不安装扩展，那么 `PHP` 无法操控 `Memcached`。

同样如果安装了 `PHP Memcached` 扩展（PHP Memcache 和 PHP memcached 两者选择一个)，但是没有安装 `Memcached` 服务端，那么这个就无法使用。

只有同时安装了 `Memcached` 服务端和 PHP 客户端扩展才可以提高动态网站性能。

## 具体应用

- 确保 `PHP` 安装并引入了 `memcached` 扩展（可能要重启 php-fpm）；确保 `Memcached` 已正常运行（略）

### 本地测试

> windows下安装PHP7.0/7.1的Memcache扩展: <u>[nono303/PHP7-memcache-dll](https://github.com/nono303/PHP7-memcache-dll)</u>

```
    <?php
    
    $memcache = new Memcache;
    $memcache->connect('localhost', 11211) or die ("Could not connect");
    
    $version = $memcache->getVersion();
    echo "Memcache Server's version: ".$version."<br/>\n";
    
    $tmp_object = new stdClass;
    $tmp_object->str_attr = 'test';
    $tmp_object->int_attr = 123;
    
    $memcache->set('key', $tmp_object, false, 10) or die ("Failed to save data at the server");
    
    echo "Store data in the cache (data will expire in 10 seconds)<br/>\n";
    
    $get_result = $memcache->get('key');
    echo "Data from the cache:<br/>\n";
    
    var_dump($get_result);
```

### 在`Lumen`中应用

- 创建并配置 `config/cache.php`：

```
    <?php
    
    return [
        'default' => 'memcached',
    
        'stores' => [
            'memcached' => [
                'driver' => 'memcached',
                'servers' => [
                    [
                        'host' => env('MEMCACHED_HOST', '127.0.0.1'),
                        'port' => env('MEMCACHED_PORT', 11211),
                        'weight' => 100,
                    ],
                ],
            ],
        ],
        
    ];
```

- 配置`.env`

```
    CACHE_DRIVER=memcached
    MEMCACHED_HOST=172.17.0.11
```

- `bootstrap/app.php` 中引入配置和 `memcached.connector`：

```
    $app->configure('cache');
    
    // !!! 不配会坑
    $app->singleton(
        'memcached.connector', Illuminate\Cache\MemcachedConnector::class
    );
```

### 在`Laravel`中重新封装

- `app/Contract/Service/Cachable.php`

```
    <?php
    
    // Cache Service Interface
    // @caoxl
    
    namespace App\Contract\Service;
    
    interface Cachable
    {
        // Get cached value by given key
        public function get(string $key) : string;
    
        // Cache given value by key for seconds
        public function set(string $key, string $value, int $seconds) : Cachable;
    
        // Get cached value by given key
        // And delete that cache when fetch it successfully
        public function flush(string $key) : string;
    
        // Delete cached value by given key
        public function delete(string $key) : bool;
    
        public function incrementToday(string $key, int $step = 1) : int;
    }
```

- `app/Service/Cache/Laravel.php`

```
    <?php
    
    // Cache service re-packing from Laravel
    // @caoxl
    
    namespace App\Services\Cache;
    
    use Illuminate\Support\Facades\Cache;
    use App\Contract\Service\Cachable;
    
    class Laravel implements Cachable
    {
        public function get(string $key): string
        {
            return (string) Cache::get($key);
        }
    
        public function set(string $key, string $value, int $seconds): Cachable
        {
            Cache::put($key, $value, ($seconds/60));
    
            return $this;
        }
    
        public function flush(string $key): string
        {
            $value = null;
    
            if (Cache::has($key)) {
                $value = Cache::get($key);
                Cache::forget($key);
            }
    
            return (string) $value;
        }
    
        public function delete(string $key): bool
        {
            if (Cache::has($key)) {
                return Cache::forget($key);
            }
    
            return true;
        }
        public function incrementToday(string $key, int $step = 1): int
        {
            $times   = intval(Cache::get($key));
            $times  += $step;
            $minutes = ($this->seconds_left_today() / 60);
    
            Cache::put($key, $times, $minutes);
    
            return $times;
        }
    
        private function seconds_left_today() : int
        {
            $tomorrow = strtotime(date('Y-m-d', strtotime('+1 day')));
    
            return $tomorrow - time();
        }
    }
```

注意：这里如果不事先加载单例 `MemcachedConnector` ，则会报如下错误：

> lumen.ERROR: ReflectionException: Class memcached.connector does not exist in /data/www/vendor/illuminate/container/Container.php:729`

- 如果是生产环境?

生产环境一般开启了 autoload 等缓存了的，为确保正确加载，部署时要清理下缓存先：

```
    php artisan config:clear
    composer dump-autoload
    
    # Lumen 默认没有以下命令
    # php artisan optimize
    # php artisan clear-compiled
```

> `memcached` 默认的过期时间则 不能大于 `30` 天！

# `Redis`

- 安装`redis`, 使用`composer`即可:

```
    composer require illuminate/redis
    composer require predis/predis
```

- 或者

```
        "require": {
            "php": ">=7.1.3",
            "illuminate/redis": "5.5.*",
            "laravel/lumen-framework": "5.5.*",
            "predis/predis": "^1.1",
        },
```

- 注册`redis`类

```
    $app->register(Illuminate\Redis\RedisServiceProvider::class);
```

- 在要使用`redis`的地方即可

```
    use Illuminate\Support\Facades\Redis;
```

- `redis`常用命令

```
    <?php
    
    // Redis Reference Resources
    // 这个文件仅供忘记Redis命令时参考,使用Redis无需重新封装.
    // @caoxl
    
    namespace App\Traits;
    
    class Redis
    {
        public function set($key, $val)
        {
            Redis::set($key, $val);
        }
    
        public function setex($key, $exp, $val)
        {
            Redis::setex($key, $exp, $val);
        }
    
        public function get($key)
        {
            return Redis::get($key);
        }
    
        public function expire($key, $exp = 180)
        {
            Redis::expire($key, $exp);
        }
    
        public function incr($key)
        {
            Redis::incr($key);
        }
    
        public function del($key)
        {
            Redis::del($key);
        }
    
        public function exists($key)
        {
            return Redis::exists($key);
        }
    }
```


# 参考

- [在 Windows 10 64 下安装 Memcached，安装 PHP 7.0.22 的 Memcache 扩展](http://www.shuijingwanwq.com/2017/09/11/1906/)