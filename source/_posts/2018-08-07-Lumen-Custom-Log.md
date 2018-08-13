---
title: 基于Lumen-自定义日志文件名
date: 2018-08-07 11:23:26
categories: Lumen
tags: [Lumen, 自定义日志]
---

> `Lumen` 中默认的文件名是被硬编码在 `Application` 类中了的，要实现这样的效果，必须重写 `Laravel\Lumen\Application@getMonologHandler()` 方法。

<!-- more -->

- 源文件 `/bootstrap/app.php`

```
    $app = new Laravel\Lumen\Application(
        realpath(__DIR__.'/../')
    );
```

- 修改后 `/bootstrap/app.php`

```
    $app = new App\Application(
        realpath(__DIR__.'/../')
    );
```

- 新增文件: `app/Application.php`

```
    <?php
    
    // Custom default or hard-coded behavior of lumen
    // @caoxl
    
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

> 在`/storage/logs/`即可看到以类似 `2018-08-07.log`命名的日志文件
