---
title: Laravel 5.5 「定义函数」
date: 2018-01-31 15:38:21
categories: Laravel
tags: [PHP, Laravel, DingoAPI, Framework]
---

以下函数,在工作中学习大神的一些自定义函数

> 废话不多说,先上源码,后续慢慢分解

> - 以下代码运行在 Laravel 5.5 / PHP 7.1+ / MySQL 5.6+
> > 代码位置: bootstrap/functions

<!--more-->

# 源码

- `bootstrap/functions.php`

```
    <?php
    
    // Application logic level user-defined global functions
    // !!! Load after laravel base components and before application
    
    if (! function_exists('fe')) {
        function fe(string $name) {
            return function_exists($name);
        }
    }
    
    if (! fe('ci_equal')) {
        function ci_equal($foo = null, $bar = null) : bool {
            if (!$foo || !$bar || !is_scalar($foo) || !is_scalar($bar)) {
                return false;
            }
    
            return (strtolower($foo) === strtolower($bar));
        }
    }
    
    if (! fe('db_driver')) {
        function db_driver(string $driver = null) {
            $conn    = config('database.default');
            $_driver = config("database.connections.{$conn}.driver");
    
            return $driver ? (ci_equal($driver, $_driver)) : $_driver;
        }
    }
    
    if (! fe('is_mariadb')) {
        function is_mariadb() {
            $dbver   = \DB::select('SHOW VARIABLES LIKE "version_comment"');
            return  preg_match('/(mariadb)/ui', $dbver[0]->Value);
        }
    }
    
    if (! fe('mysql_ver_below')) {
        function mysql_ver_below($compare = 5.7) {
            if (! db_driver('mysql')) {
                return false;
            }
            
            $compare = is_mariadb() ? 11 : $compare;
    
            return (mysql_ver() < $compare);
        }
    }
    
    if (! fe('mysql_ver')) {
        function mysql_ver() {
            $ver = \DB::select('SELECT VERSION() AS VER');
    
            return floatval($ver[0]->VER ?? 0);
        }
    }
    
    if (! fe('seed_one')) {
        function seed_one(string $table, array $data, string $pk = 'id') : int {
            if (\Schema::hasTable($table)) {
                if ($data) {
                    $item = $data;
                    $data = [$item];
                    seeder_filter($item, $data, 0);
                    if (\DB::table($table)->where($item)->count()) {
                        return intval(\DB::table($table)
                            ->select($pk)
                            ->where($item)
                            ->first()
                            ->$pk ?? 0
                        );
                    } else {
                        echo 'Seeding table: ', $table, PHP_EOL;
    
                        return intval(\DB::table($table)->insertGetId($data[0]));
                    }
                }
            }
    
            return 0;
        }
    }
    
    if (! fe('empty_safe')) {
        function empty_safe($var) {
            if (is_numeric($var)) {
                return false;
            }
            
            return empty($var);
        }
    }
    
    if (! fe('seeder_filter')) {
        function seeder_filter(&$item, &$data = null, $idx = null) {
            array_walk($item, function (&$v, $k) use (
                &$data, &$item, $idx
            ) {
                if (is_array($v) && isset($v[0])) {
                    if ($data && !empty_safe($idx)) {
                        $data[$idx][$k] = (string) $v[0];
                    }
    
                    unset($item[$k]);
                }
            });
        }
    }
    
    if (! fe('seed_once')) {
        function seed_once(string $table, array $data) {
            if (\Schema::hasTable($table)) {
                foreach ($data as $idx => $item) {
                    if (is_array($item)) {
                        seeder_filter($item, $data, $idx);
    
                        if (\DB::table($table)->where($item)->count()) {
                            unset($data[$idx]);
                        }
                    }
                }
    
                echo 'Seeding table: ', $table, PHP_EOL;
    
                if ($data) {
                    \DB::table($table)->insert($data);
                }
            } else {
                echo 'Missing table: ', $table, ', skipped.', PHP_EOL;
            }
        }
    }
    
    if (! fe('excp')) {
        function excp($msg, $err = 500) {
            throw new \Exception($msg, $err);
        }
    }
    
    if (! fe('load_phps')) {
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
    }
    
    if (! fe('route_path')) {
        function route_path(string $name = null, bool $file = false) {
            $argv  = $name ? "/{$name}" : null;
            $argv .= $file ? '.php' : null;
    
            return base_path("routes{$argv}");
        }
    }
    
    if (! fe('dingo')) {
        function dingo() {
            return app('Dingo\Api\Routing\Router');
        }
    }
    
    if (! fe('auth_response')) {
        function auth_response(string $token) {
            return (
                new \Illuminate\Http\Resources\Json\Resource(
                    new \App\Models\Authorization($token)
                )
            )
            ->response()
            ->setStatusCode(201);
        }
    }
    
    if (! fe('tz')) {
        function tz() {
            return env('APP_TIMEZONE', 'Asia/Shanghai');
        }
    }
    
    if (! fe('fdate')) {
        function fdate($ts = null, $tz = null) {
            date_default_timezone_set(tz());
    
            $ts = $ts ?? time();
    
            return date('Y-m-d H:i:s', $ts);
        }
    }
    
    if (! fe('mobile_regex_cn')) {
        function mobile_regex_cn() : string {
            return '/^1[3-9]\d{9}$/u';
        }
    }
    
    if (! fe('is_mobile')) {
        function is_mobile(string $mobile) : bool {
            // return (true
            //     && ($mobile == intval($mobile))
            //     && (1300000000 <= $mobile)
            //     && ($mobile <= 19999999999)
            // );
    
            return ((bool) preg_match(mobile_regex_cn(), $mobile));
        }
    }
    
    if (! fe('service_path')) {
        function service_path(string $subpath = null) {
            return base_path("app/Service/{$subpath}");
        }
    }
    
    if (! fe('service_exist')) {
        function service_exist(string $key = null) {
            if (false
                || (! ($key = (strtolower($key))))
                || (! ($path = array_filter(explode('.', $key))))
            ) {
                return false;
            }
    
            array_walk($path, function (&$item) {
                if ($item && is_string($item)) {
                    $item = ucfirst(strtolower($item));
                }
            });
    
            $_path = service_path(implode('/', $path));
    
            if (true
                && (false
                    || is_file("{$_path}.php")
                    || (true
                        && is_dir($_path)
                        && ($default = ucfirst(config("service.{$key}.default")))
                        && (is_file("{$_path}/{$default}.php"))
                    )
                )
            ) {
                return [$path, ($default ?? null)];
            }
    
            return false;
        }
    }
    
    if (! fe('service')) {
        function service(string $key, string $child = null) {
            if ($items = service_exist("{$key}.{$child}")) {
                list($path, $default) = $items;
                array_unshift($path, 'Service');
                array_unshift($path, 'App');
    
                if ($default) {
                    array_push($path, $default);
                }
                
                if (class_exists($service = implode('\\', $path))) {
                    return new $service;
                }
            }
    
            excp("Service not exists: {$key}.");
        }
    }
    
    if (! fe('api_response')) {
        function api_response(
            int $err = null,
            string $msg = null,
            $data = null,
            array $headers = []
        ) {
            return response(
                apimat($err, $msg, $data),
                $err,
                $headers
            );
        }
    }
    
    if (! fe('apimat')) {
        function apimat(int $err = null, string $msg = null, $data = null) {
            $format = [
                'status_code' => ($err ?? 200),
                'message' => ($msg ?? 'ok'),
            ];
    
            if ($data) {
                $format['__data'] = $data;
            }
    
            return $format;
        }
    }
    
    if (! fe('validate')) {
        function validate(array $data = [], array $rules = []) {
            $validator = validator($data, $rules);
    
            if ($validator->fails()) {
                return $validator->errors()->first();
            }
    
            return true;
        }
    }
    
    if (! fe('gen_rand')) {
        function gen_rand(int $length = null, string $format = null): string
        {
            $length = $length ?? 4;
            $format = $type   ?? 'int';
    
            if (!is_integer($length) || (0 > $length)) {
                excp('Checkcode length must be an integer over 0.');
            }
    
            $chars = $pureNum = str_split('0123456789');
    
            if (ci_equal($format, 'str')) {
                $charLower = 'abcdefghijklmnopqrstuvwxyz';
                $charUpper = strtoupper($charLower);
                $chars     = array_merge(
                    $chars,
                    str_split($charLower.$charUpper)
                );
            }
    
            $charsLen = count($chars) - 1;
            
            $code = '';
            for ($i=0; $i<$length; ++$i) {
                $code .= $chars[mt_rand(0, $charsLen)];
            }
    
            return $code;
        }
    }
    
    if (! fe('ucid')) {
        function ucid() {
            $mt = LARAVEL_START ?? microtime(true);
    
            return gen_rand().md5(getmypid()).'_'.$mt;
        }
    }
    
    if (! fe('seconds_left_today')) {
        function seconds_left_today() : int {
            $tomorrow = strtotime(date('Y-m-d', strtotime('+1 day')));
    
            return $tomorrow - time();
        }
    }
    
    if (! fe('trade_no')) {
        function trade_no(int $mid = 0, int $env = 0) {
            $env     = str_pad(($env % 10), 2, '0', STR_PAD_LEFT);
            $mid     = str_pad(($mid % 10), 2, '0', STR_PAD_LEFT);
            $rand    = str_pad(mt_rand(0, 99), 2, '0', STR_PAD_LEFT);
            $postfix = mb_substr(microtime(), 2, 6);
    
            return date('ymdHis').$env.$mid.$rand.$postfix;
        }
    }
    
    if (! fe('find_or_create_user_by_mobile')) {
        function find_or_create_user_by_mobile($mobile) {
            return (new \App\User())->findOrCreateByUsername($mobile);
        }
    }
    
    if (! fe('find_user_by_mobile')) {
        function find_user_by_username($username) {
            return \App\User::whereUsername($username)->first();
        }
    }
    
    if (! fe('phash')) {
        function phash(string $pswd) {
            return password_hash($pswd, PASSWORD_DEFAULT);
        }
    }
    
    if (! fe('api_route')) {
        function api_route(string $name, string $ver) {
            return app('Dingo\Api\Routing\UrlGenerator')
            ->version($ver)
            ->route($name);
        }
    }
    
    if (! fe('httpclient')) {
        function httpclient() {
            return new \GuzzleHttp\Client();
        }
    }
    
    if (! fe('auth_response_from_user')) {
        function auth_response_from_user(\App\Models\User $user = null) {
            if ((! $user) || (! ($token = \Auth::fromUser($user)))) {
                return api_response(401, 'Bad user credentials');
            }
    
            return auth_response($token);
        }
    }
    
    if (! fe('ispint')) {
        function ispint($num, bool $zero = false) {
            $zero = $zero ? -1 : 0;
            $_num = $num;
    
            return (true
                && (! empty_safe($num))
                && is_numeric($num)
                && (($num = intval($num)) == $_num)
                && ($num > $zero)
            );
        }
    }
    
    if (! fe('client_ip')) {
        function client_ip() {
            foreach ([
                'HTTP_CLIENT_IP',
                'HTTP_X_FORWARDED_FOR',
                'HTTP_X_FORWARDED',
                'HTTP_FORWARDED_FOR',
                'HTTP_FORWARDED',
                'REMOTE_ADDR',
            ] as $residence) {
                if ($ip = (getenv($residence) ?? ($_SERVER[$residence] ?? null))) {
                    return $ip;
                }
            }
    
            return 'UNKNOWN';       
        }
    }
    if (! fe('is_intstr')) {
        function is_intstr(string $var) {
            return (true
                && is_numeric($var)
                && ($var == ($_var = intval($var)))
            ) ? $_var : false;
        }
    }
    if (! fe('strtodate')) {
        function strtodate(string $time, bool $zerofill = true) {
            $time = is_intstr($time) ? $time : strtotime($time);
            $day  = date('Y-m-d', $time);
    
            return $zerofill ? "{$day} 00:00:00" : $day;
        }
    }
    if (! fe('days_range_before')) {
        function days_range_before(string $date, int $days) {
            $ts = time();
    
            return (true
                && ($days > 0)
                && ($days >= day_diff($date, $ts))
                && (strtotime(strtodate($date)) <= $ts)
            );
        }
    }
    if (! fe('day_diff')) {
        function day_diff(string $start, string $end, int $diff = null) {
            if (!$start || !$end) {
                return false;
            }
    
            $_diff = intval(
                (new \Datetime(strtodate($end)))
                ->diff(new \DateTime(strtodate($start)))
                ->format('%a')
            );
    
            return is_null($diff) ? $_diff : ($diff === $_diff);
        }
    }
    if (! fe('is_same_day')) {
        function is_same_day(string $day1, string $day2 = null) : bool {
            if ((! $day1) || (strtotime($day1) < 1)) {
                return false;
            }
    
            $day2 = $day2 ?? time();
    
            return (strtodate($day1, false) === strtodate($day2, false));
        }
    }
    if (! fe('arr2xml')) {
        function arr2xml(array $array) : string {
            $xml     = '';
            $arr2xml = function (
                array $array, string &$xml, callable $arr2xml
            ): string {
                foreach ($array as $key => &$val) {
                    if (is_array($val)) {
                        $_xml = '';
                        $val  = $arr2xml($val, $_xml, $arr2xml);
                    }
                    $xml .= "<{$key}>{$val}</{$key}>";
                }
    
                unset($val);
    
                return $xml;
            };
    
            $_xml = $arr2xml($array, $xml, $arr2xml);
    
            unset($xml);
            
            return '<?xml version="1.0" encoding="utf-8"?><xml>'.$_xml.'</xml>';
        }
    }
    if (! fe('xml2arr')) {
        function xml2arr(string $xml) : array {
            if (! extension_loaded('libxml')) {
                excp('PHP extension missing: libxml');
            }
    
            return (array) json_decode(json_encode(simplexml_load_string(
                $xml,
                'SimpleXMLElement',
                LIBXML_NOCDATA
            )), true);
        }
    }
    if (! fe('lang')) {
        function lang(string $lang) : array {
            $type = env('i18n.lang.type', 'php');
            $path = env('i18n.lang.path', resource_path('lang'));
    
            if (! is_dir($_path = ($path.'/'.$lang))) {
                return [];
            }
    
            if ($lang = ($GLOBALS['__APP_LANG_KV'] ?? false)) {
                return (array) $lang;
            }
    
            $lang = [];
            if (ci_equal('php', $type)) {
                load_phps($_path, function ($file) use (&$lang) {
                    $_lang = include $file->getPathname();
                    array_push($lang, $_lang);
                });
    
                return $GLOBALS['__APP_LANG_KV'] = ($lang[0] ?? []);
            }
    
            // TODO
            // Support any K-V language package
    
            return [];
        }
    }
    if (! fe('i18n')) {
        function i18n(string $msg, array $context = null, string $lang = 'zh') {
            $_msg = explode(':', $msg);
            $key  = trim($_msg[0] ?? '');
            $args = trim($_msg[1] ?? '');
            $argv = false;
    
            if ($args) {
                if ($context) {
                    $argv = preg_replace_callback(
                        '/(\{(\w+)\})/ui',
                        function ($matches) use ($context) {
                            $match = $matches[2] ?? 'NIL';
    
                            if ($replace = ($context[$match] ?? null)) {
                                return $replace;
                            }
    
                            return strtoupper($match);
                        },
                        $args
                    );
                } else {
                    $argv = strtoupper($args);
                }
            }
    
            $lang  = lang($lang);
            $lval  = $lang[strtoupper($key)] ?? strtoupper($key);
            $lval .= $argv ? (': '.$argv) : '';
    
            return $lval;
        }
    }
    if (! fe('cny_yuan2fen')) {
        function cny_yuan2fen(float $yuan) : int {
            $fen = explode('.', $yuan*100);
    
            return intval($fen[0] ?? 0);
        }
    }
    if (! fe('member_level_exists')) {
        function member_level_exists(string $level) {
            return \App\Models\MemberLevel::where('key', $level)->first();
        }
    }
    if (! fe('get_member_binding')) {
        function get_member_binding(int $member, string $bind, string $attr) {
            $binding = '';
            if ($member = \App\Models\Member::find($member)) {
                $binding = $member->getUniqueBinding($bind, $attr);
            }
    
            return $binding;
        }
    }
    if (! fe('is_closure')) {
        function is_closure($var) : bool {
            return $var instanceof \Closure;
        }
    }
    if (! fe('is_model')) {
        function is_model($var) : bool {
            return $var instanceof \Illuminate\Database\Eloquent\Model;
        }
    }
```

> 看项目需要,选取自定义函数


# 源码分解

## `fe`

```
    if (! function_exists('fe')) {
        function fe(string $name) {
            return function_exists($name);
        }
    }
```
- `function_exists` — 如果给定的函数已经被定义就返回 TRUE

**该函数用于创建自定义函数**

## `ci_equal`

```
    if (! fe('ci_equal')) {
        function ci_equal($foo = null, $bar = null) : bool {
            if (!$foo || !$bar || !is_scalar($foo) || !is_scalar($bar)) {
                return false;
            }
    
            return (strtolower($foo) === strtolower($bar));
        }
    }
```

**注意** :bool 写法是PHP7的写法,要求 `PHP 7.1+`

- `is_scalar` — 检测变量是否是一个标量

**该函数用于作判断字符串是否一样**

## `db_driver`

```
    if (! fe('db_driver')) {
        function db_driver(string $driver = null) {
            $conn    = config('database.default');
            $_driver = config("database.connections.{$conn}.driver");
    
            return $driver ? (ci_equal($driver, $_driver)) : $_driver;
        }
    }
```

- `config` - 获取配置文件中的参数

**该函数用于 判断数据库的驱动**

## `is_mariadb`

```
    if (! fe('is_mariadb')) {
        function is_mariadb() {
            $dbver   = \DB::select('SHOW VARIABLES LIKE "version_comment"');
            return  preg_match('/(mariadb)/ui', $dbver[0]->Value);
        }
    }
```

- `preg_match` — 执行匹配正则表达式

**该函数用于 判断是否 Maridb**

## `mysql_ver_below`

> ver 版本

```
    if (! fe('mysql_ver_below')) {
        function mysql_ver_below($compare = 5.7) {
            if (! db_driver('mysql')) {
                return false;
            }
            
            $compare = is_mariadb() ? 11 : $compare;
    
            return (mysql_ver() < $compare);
        }
    }
```

**该函数用于 比较MYSQL版本 默认5.7**

## `mysql_ver`

```
    if (! fe('mysql_ver')) {
        function mysql_ver() {
            $ver = \DB::select('SELECT VERSION() AS VER');
    
            return floatval($ver[0]->VER ?? 0);
        }
    }
```

**该函数用于 获取MYSQL版本**

## `seed_one`

```
    if (! fe('seed_one')) {
        function seed_one(string $table, array $data, string $pk = 'id') : int {
            if (\Schema::hasTable($table)) {
                if ($data) {
                    $item = $data;
                    $data = [$item];
                    seeder_filter($item, $data, 0);
                    if (\DB::table($table)->where($item)->count()) {
                        return intval(\DB::table($table)
                            ->select($pk)
                            ->where($item)
                            ->first()
                            ->$pk ?? 0
                        );
                    } else {
                        echo 'Seeding table: ', $table, PHP_EOL;
    
                        return intval(\DB::table($table)->insertGetId($data[0]));
                    }
                }
            }
    
            return 0;
        }
    }
```

**该函数用于 数据填充 一条数据**

## `empty_safe`

```
    if (! fe('empty_safe')) {
        function empty_safe($var) {
            if (is_numeric($var)) {
                return false;
            }
            
            return empty($var);
        }
    }
```

- `is_numeric` — 检测变量是否为数字或数字字符串
- `empty` — 检查一个变量是否为空

**该函数用于 检查一个变量** 


## `seeder_filter`

```
    if (! fe('seeder_filter')) {
        function seeder_filter(&$item, &$data = null, $idx = null) {
            array_walk($item, function (&$v, $k) use (
                &$data, &$item, $idx
            ) {
                if (is_array($v) && isset($v[0])) {
                    if ($data && !empty_safe($idx)) {
                        $data[$idx][$k] = (string) $v[0];
                    }
    
                    unset($item[$k]);
                }
            });
        }
    }
```

- `array_walk` — 使用用户自定义函数对数组中的每个元素做回调处理
- `unset` — 释放给定的变量

**该函数用于 数据过滤**

## `seed_once`

```
    if (! fe('seed_once')) {
        function seed_once(string $table, array $data) {
            if (\Schema::hasTable($table)) {
                foreach ($data as $idx => $item) {
                    if (is_array($item)) {
                        seeder_filter($item, $data, $idx);
    
                        if (\DB::table($table)->where($item)->count()) {
                            unset($data[$idx]);
                        }
                    }
                }
    
                echo 'Seeding table: ', $table, PHP_EOL;
    
                if ($data) {
                    \DB::table($table)->insert($data);
                }
            } else {
                echo 'Missing table: ', $table, ', skipped.', PHP_EOL;
            }
        }
    }
```

**该函数用于 数据填充**

## `excp`

```
    if (! fe('excp')) {
        function excp($msg, $err = 500) {
            throw new \Exception($msg, $err);
        }
    }
```

**该函数用于 抛异常**

## `load_phps`

```
    if (! fe('load_phps')) {
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
    }
```

- `file_exists` — 检查文件或目录是否存在

文件函数:
 - $file->isFile()
 - $file->getExtension()
 - $file->getPathname()
 - $file->isDir()
 - $file->getBasename()

**该函数用于 加载文件**

## `route_path`

```
    if (! fe('route_path')) {
        function route_path(string $name = null, bool $file = false) {
            $argv  = $name ? "/{$name}" : null;
            $argv .= $file ? '.php' : null;
    
            return base_path("routes{$argv}");
        }
    }
```

**该函数用于 获取路由路径**

## `dingo`

```
    if (! fe('dingo')) {
        function dingo() {
            return app('Dingo\Api\Routing\Router');
        }
    }
```

**该函数用于 添加Dingo代码**

## `auth_response`

```
    if (! fe('auth_response')) {
        function auth_response(string $token) {
            return (
                new \Illuminate\Http\Resources\Json\Resource(
                    new \App\Models\Authorization($token)
                )
            )
            ->response()
            ->setStatusCode(201);
        }
    }
```

**该函数用于 访问权限响应**

## `tz`

```
    if (! fe('tz')) {
        function tz() {
            return env('APP_TIMEZONE', 'Asia/Shanghai');
        }
    }
```

**该函数用于 将TIMEZONE 设置为 'Asia/Shanghai'**


## `fdate`

```
    if (! fe('fdate')) {
        function fdate($ts = null, $tz = null) {
            date_default_timezone_set(tz());
    
            $ts = $ts ?? time();
    
            return date('Y-m-d H:i:s', $ts);
        }
    }
```

- `date_default_timezone_set` — 设定用于一个脚本中所有日期时间函数的默认时区

**该函数用于 返回格式化的时间**


## `mobile_regex_cn`

```
    if (! fe('mobile_regex_cn')) {
        function mobile_regex_cn() : string {
            return '/^1[3-9]\d{9}$/u';
        }
    }
```

**该函数用于 检查手机号码的格式 (国内号码)**

## `is_mobile`

```
    if (! fe('is_mobile')) {
        function is_mobile(string $mobile) : bool {
            // return (true
            //     && ($mobile == intval($mobile))
            //     && (1300000000 <= $mobile)
            //     && ($mobile <= 19999999999)
            // );
    
            return ((bool) preg_match(mobile_regex_cn(), $mobile));
        }
    }
```

**该函数用于 判断是否是手机号码**

## `service_path`

```
    if (! fe('service_path')) {
        function service_path(string $subpath = null) {
            return base_path("app/Service/{$subpath}");
        }
    }
```

**该函数用于 获取 `Service` 目录**

## `service_exist`

```
    if (! fe('service_exist')) {
        function service_exist(string $key = null) {
            if (false
                || (! ($key = (strtolower($key))))
                || (! ($path = array_filter(explode('.', $key))))
            ) {
                return false;
            }
    
            array_walk($path, function (&$item) {
                if ($item && is_string($item)) {
                    $item = ucfirst(strtolower($item));
                }
            });
    
            $_path = service_path(implode('/', $path));
    
            if (true
                && (false
                    || is_file("{$_path}.php")
                    || (true
                        && is_dir($_path)
                        && ($default = ucfirst(config("service.{$key}.default")))
                        && (is_file("{$_path}/{$default}.php"))
                    )
                )
            ) {
                return [$path, ($default ?? null)];
            }
    
            return false;
        }
    }
```

- `strtolower` — 将字符串转化为小写
- `array_filter` — 用回调函数过滤数组中的单元
- `array_walk` — 使用用户自定义函数对数组中的每个元素做回调处理
- `is_file` — 判断给定文件名是否为一个正常的文件
- `is_dir` — 判断给定文件名是否是一个目录

**该函数用于 判断 `Service` 是否存在**

## `service`

```
    if (! fe('service')) {
        function service(string $key, string $child = null) {
            if ($items = service_exist("{$key}.{$child}")) {
                list($path, $default) = $items;
                array_unshift($path, 'Service');
                array_unshift($path, 'App');
    
                if ($default) {
                    array_push($path, $default);
                }
                
                if (class_exists($service = implode('\\', $path))) {
                    return new $service;
                }
            }
    
            excp("Service not exists: {$key}.");
        }
    }
```

- `list` — 把数组中的值赋给一组变量
- `array_unshift` — 在数组开头插入一个或多个单元
- `array_push` — 将一个或多个单元压入数组的末尾（入栈）
- `class_exists` — 检查类是否已定义

**该函数用于 实例化Service**


## `api_response`

```
    if (! fe('api_response')) {
        function api_response(
            int $err = null,
            string $msg = null,
            $data = null,
            array $headers = []
        ) {
            return response(
                apimat($err, $msg, $data),
                $err,
                $headers
            );
        }
    }
```

**该函数用于 API响应**

## `apimat`

```
    if (! fe('apimat')) {
        function apimat(int $err = null, string $msg = null, $data = null) {
            $format = [
                'status_code' => ($err ?? 200),
                'message' => ($msg ?? 'ok'),
            ];
    
            if ($data) {
                $format['__data'] = $data;
            }
    
            return $format;
        }
    }
```

**该函数用于 格式化 API响应数据**

## `validate`

```
    if (! fe('validate')) {
        function validate(array $data = [], array $rules = []) {
            $validator = validator($data, $rules);
    
            if ($validator->fails()) {
                return $validator->errors()->first();
            }
    
            return true;
        }
    }
```

**该函数用于 封装了 `Laravel` 的 `Validate` 数据验证**

## `gen_rand`

```
    if (! fe('gen_rand')) {
        function gen_rand(int $length = null, string $format = null): string
        {
            $length = $length ?? 4;
            $format = $type   ?? 'int';
    
            if (!is_integer($length) || (0 > $length)) {
                excp('Checkcode length must be an integer over 0.');
            }
    
            $chars = $pureNum = str_split('0123456789');
    
            if (ci_equal($format, 'str')) {
                $charLower = 'abcdefghijklmnopqrstuvwxyz';
                $charUpper = strtoupper($charLower);
                $chars     = array_merge(
                    $chars,
                    str_split($charLower.$charUpper)
                );
            }
    
            $charsLen = count($chars) - 1;
            
            $code = '';
            for ($i=0; $i<$length; ++$i) {
                $code .= $chars[mt_rand(0, $charsLen)];
            }
    
            return $code;
        }
    }
```

- `str_split` — 将字符串转换为数组
- `strtoupper` — 将字符串转化为大写
- `array_merge` — 合并一个或多个数组

**该函数用于 生成随机数/随机字符串**

## `ucid`

```
    if (! fe('ucid')) {
        function ucid() {
            $mt = LARAVEL_START ?? microtime(true);
    
            return gen_rand().md5(getmypid()).'_'.$mt;
        }
    }
```

**该函数用于 生成自定义的 `UCID` 可作订单号之类**

## `seconds_left_today`

```
    if (! fe('seconds_left_today')) {
        function seconds_left_today() : int {
            $tomorrow = strtotime(date('Y-m-d', strtotime('+1 day')));
    
            return $tomorrow - time();
        }
    }
```

- `strtotime` — 将任何字符串的日期时间描述解析为 Unix 时间戳

**该函数用于 距离明天这个时候 还有多少秒**

## `trade_no`

```
    if (! fe('trade_no')) {
        function trade_no(int $mid = 0, int $env = 0) {
            $env     = str_pad(($env % 10), 2, '0', STR_PAD_LEFT);
            $mid     = str_pad(($mid % 10), 2, '0', STR_PAD_LEFT);
            $rand    = str_pad(mt_rand(0, 99), 2, '0', STR_PAD_LEFT);
            $postfix = mb_substr(microtime(), 2, 6);
    
            return date('ymdHis').$env.$mid.$rand.$postfix;
        }
    }
```

- `str_pad` — 使用另一个字符串填充字符串为指定长度
  - `STR_PAD_BOTH` - 填充字符串的两侧。如果不是偶数，则右侧获得额外的填充。
  - `STR_PAD_LEFT` - 填充字符串的左侧。
  - `STR_PAD_RIGHT` - 填充字符串的右侧。默认。
- `mt_rand` — 生成更好的随机数
- `mb_substr` — 获取部分字符串
- `microtime` — 返回当前 Unix 时间戳和微秒数

**该函数用于 生成一个 交易/订单号**

## `find_or_create_user_by_mobile`

```
    if (! fe('find_or_create_user_by_mobile')) {
        function find_or_create_user_by_mobile($mobile) {
            return (new \App\User())->findOrCreateByUsername($mobile);
        }
    }
```

**该函数用于 通过Mobile 获取用户信息/创建用户**

## `find_user_by_mobile`

```
    if (! fe('find_user_by_mobile')) {
        function find_user_by_username($username) {
            return \App\User::whereUsername($username)->first();
        }
    }
```

**该函数用于 查询用户 通过Mobile**

## `phash`

```
    if (! fe('phash')) {
        function phash(string $pswd) {
            return password_hash($pswd, PASSWORD_DEFAULT);
        }
    }
```

- `password_hash()` - 使用足够强度的单向散列算法创建密码的散列（hash）。

当前支持的算法：

  - `PASSWORD_DEFAULT` - 使用 bcrypt 算法 (PHP 5.5.0 默认)。 注意，该常量会随着 PHP 加入更新更高强度的算法而改变。 所以，使用此常量生成结果的长度将在未来有变化。 因此，数据库里储存结果的列可超过60个字符（最好是255个字符）。
  
  - `PASSWORD_BCRYPT` - 使用 CRYPT_BLOWFISH 算法创建散列。 这会产生兼容使用 "$2y$" 的 crypt()。 结果将会是 60 个字符的字符串， 或者在失败时返回 FALSE。
  
    - `salt(string)` - 手动提供散列密码的盐值（salt）。这将避免自动生成盐值（salt）。
      
      省略此值后，`password_hash()` 会为每个密码散列自动生成随机的盐值。这种操作是有意的模式。
    - `cost (integer)` - 代表算法使用的 cost。crypt() 页面上有 cost 值的例子。
      
      省略时，默认值是 10。 这个 cost 是个不错的底线，但也许可以根据自己硬件的情况，加大这个值。
     
  - `PASSWORD_ARGON2I` - 使用 Argon2 散列算法创建散列。

  
**该函数用于 生成 Hash密码**

## `api_route`

```
    if (! fe('api_route')) {
        function api_route(string $name, string $ver) {
            return app('Dingo\Api\Routing\UrlGenerator')
            ->version($ver)
            ->route($name);
        }
    }
```

**该函数用于 生成 Dingo API 路由**

## `httpclient`

```
    if (! fe('httpclient')) {
        function httpclient() {
            return new \GuzzleHttp\Client();
        }
    }
```

> PHP HTTP Client客户端---Guzzle

**该函数用于 实例化 Guzzle客户端**


## `auth_response_from_user`

```
    if (! fe('auth_response_from_user')) {
        function auth_response_from_user(\App\Models\User $user = null) {
            if ((! $user) || (! ($token = \Auth::fromUser($user)))) {
                return api_response(401, 'Bad user credentials');
            }
    
            return auth_response($token);
        }
    }
```

**该函数用于 验证用户的权限**

## `ispint`

```
    if (! fe('ispint')) {
        function ispint($num, bool $zero = false) {
            $zero = $zero ? -1 : 0;
            $_num = $num;
    
            return (true
                && (! empty_safe($num))
                && is_numeric($num)
                && (($num = intval($num)) == $_num)
                && ($num > $zero)
            );
        }
    } 
```

**该函数用于 判断 Number**

## `client_ip`

```
    if (! fe('client_ip')) {
        function client_ip() {
            foreach ([
                'HTTP_CLIENT_IP',
                'HTTP_X_FORWARDED_FOR',
                'HTTP_X_FORWARDED',
                'HTTP_FORWARDED_FOR',
                'HTTP_FORWARDED',
                'REMOTE_ADDR',
            ] as $residence) {
                if ($ip = (getenv($residence) ?? ($_SERVER[$residence] ?? null))) {
                    return $ip;
                }
            }
    
            return 'UNKNOWN';       
        }
    }
```

**该函数用于 获取客户端IP地址**

## `is_intstr`

```
    if (! fe('is_intstr')) {
        function is_intstr(string $var) {
            return (true
                && is_numeric($var)
                && ($var == ($_var = intval($var)))
            ) ? $_var : false;
        }
    }
```

**该函数用于 判断是否整型**

## `strtodate`

```
    if (! fe('strtodate')) {
        function strtodate(string $time, bool $zerofill = true) {
            $time = is_intstr($time) ? $time : strtotime($time);
            $day  = date('Y-m-d', $time);
    
            return $zerofill ? "{$day} 00:00:00" : $day;
        }
    }
```

**该函数用于 将时间转化为 标准格式的日期**

## `days_range_before`

```
    if (! fe('days_range_before')) {
        function days_range_before(string $date, int $days) {
            $ts = time();
    
            return (true
                && ($days > 0)
                && ($days >= day_diff($date, $ts))
                && (strtotime(strtodate($date)) <= $ts)
            );
        }
    }
```

**该函数用于 是否多少天前**


## `day_diff`

```
    if (! fe('day_diff')) {
        function day_diff(string $start, string $end, int $diff = null) {
            if (!$start || !$end) {
                return false;
            }
    
            $_diff = intval(
                (new \Datetime(strtodate($end)))
                ->diff(new \DateTime(strtodate($start)))
                ->format('%a')
            );
    
            return is_null($diff) ? $_diff : ($diff === $_diff);
        }
    }
```

**该函数用于 获取日期差**

## `is_same_day`

```
    if (! fe('is_same_day')) {
        function is_same_day(string $day1, string $day2 = null) : bool {
            if ((! $day1) || (strtotime($day1) < 1)) {
                return false;
            }
    
            $day2 = $day2 ?? time();
    
            return (strtodate($day1, false) === strtodate($day2, false));
        }
    }
```

**该函数用于 判断是否同一天**


## `arr2xml`

```
    if (! fe('arr2xml')) {
        function arr2xml(array $array) : string {
            $xml     = '';
            $arr2xml = function (
                array $array, string &$xml, callable $arr2xml
            ): string {
                foreach ($array as $key => &$val) {
                    if (is_array($val)) {
                        $_xml = '';
                        $val  = $arr2xml($val, $_xml, $arr2xml);
                    }
                    $xml .= "<{$key}>{$val}</{$key}>";
                }
    
                unset($val);
    
                return $xml;
            };
    
            $_xml = $arr2xml($array, $xml, $arr2xml);
    
            unset($xml);
            
            return '<?xml version="1.0" encoding="utf-8"?><xml>'.$_xml.'</xml>';
        }
    }
```

**该函数用于 数组转化为XML格式**

## `xml2arr`

```
    if (! fe('xml2arr')) {
        function xml2arr(string $xml) : array {
            if (! extension_loaded('libxml')) {
                excp('PHP extension missing: libxml');
            }
    
            return (array) json_decode(json_encode(simplexml_load_string(
                $xml,
                'SimpleXMLElement',
                LIBXML_NOCDATA
            )), true);
        }
    }
```

**该函数用于 XML格式转化成数组格式**

## `lang`

```
    if (! fe('lang')) {
        function lang(string $lang) : array {
            $type = env('i18n.lang.type', 'php');
            $path = env('i18n.lang.path', resource_path('lang'));
    
            if (! is_dir($_path = ($path.'/'.$lang))) {
                return [];
            }
    
            if ($lang = ($GLOBALS['__APP_LANG_KV'] ?? false)) {
                return (array) $lang;
            }
    
            $lang = [];
            if (ci_equal('php', $type)) {
                load_phps($_path, function ($file) use (&$lang) {
                    $_lang = include $file->getPathname();
                    array_push($lang, $_lang);
                });
    
                return $GLOBALS['__APP_LANG_KV'] = ($lang[0] ?? []);
            }
    
            // TODO
            // Support any K-V language package
    
            return [];
        }
    }
```

**该函数用于 Lang**

## `i18n`

```
    if (! fe('i18n')) {
        function i18n(string $msg, array $context = null, string $lang = 'zh') {
            $_msg = explode(':', $msg);
            $key  = trim($_msg[0] ?? '');
            $args = trim($_msg[1] ?? '');
            $argv = false;
    
            if ($args) {
                if ($context) {
                    $argv = preg_replace_callback(
                        '/(\{(\w+)\})/ui',
                        function ($matches) use ($context) {
                            $match = $matches[2] ?? 'NIL';
    
                            if ($replace = ($context[$match] ?? null)) {
                                return $replace;
                            }
    
                            return strtoupper($match);
                        },
                        $args
                    );
                } else {
                    $argv = strtoupper($args);
                }
            }
    
            $lang  = lang($lang);
            $lval  = $lang[strtoupper($key)] ?? strtoupper($key);
            $lval .= $argv ? (': '.$argv) : '';
    
            return $lval;
        }
    }
```

**该函数用于 i18n 国际化**

## `cny_yuan2fen`

```
    if (! fe('cny_yuan2fen')) {
        function cny_yuan2fen(float $yuan) : int {
            $fen = explode('.', $yuan*100);
    
            return intval($fen[0] ?? 0);
        }
    }
```

**该函数用于  人民币元转分**

## `member_level_exists`

```
    if (! fe('member_level_exists')) {
        function member_level_exists(string $level) {
            return \App\Models\MemberLevel::where('key', $level)->first();
        }
    }
```

**该函数用于 判断 会员等级是否存在(针对项目)**

## `get_member_binding`

```
    if (! fe('get_member_binding')) {
        function get_member_binding(int $member, string $bind, string $attr) {
            $binding = '';
            if ($member = \App\Models\Member::find($member)) {
                $binding = $member->getUniqueBinding($bind, $attr);
            }
    
            return $binding;
        }
    }
```

**该函数用于 获取成员绑定(针对项目)**


## `is_closure`

```
    if (! fe('is_closure')) {
        function is_closure($var) : bool {
            return $var instanceof \Closure;
        }
    }
```

**该函数用于 判断是否闭包**

## `is_model`

```
    if (! fe('is_model')) {
        function is_model($var) : bool {
            return $var instanceof \Illuminate\Database\Eloquent\Model;
        }
    }
```

**该函数用于 判断是否模型**

> 注意很多函数,相互关联.

# FAQ

