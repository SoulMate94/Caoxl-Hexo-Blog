---
title: Laravel中队列的应用 「database」
date: 2018-08-09 09:35:44
categories: Laravel
tags: [Laravel, Queue, Framework]
---

> `Laravel` 使用`database`驱动做队列

<!--more-->

# 使用`Databse`驱动步骤

1. 修改 `.env` 中的配置项 `QUEUE_DRIVER` 为 `database`

> [Laravel队列系统介绍](https://laravel-china.org/docs/laravel/5.5/queues/1324)

2. 首先要创建数据表来存储任务:

```
    php artisan queue:table
    php artisan migrate
```
3. 创建 `job` 文件:

```
    php artisan make:job SendReminderEmail
```

4. 在 `Controller` 层 `$this->dispatch(new SendRemindEmail())` 生产队列，于是 `database jobs` 中记录了最新添加的队列任务

5. 单一消费队列:

```
    php artisan queue:work
```

6. 依次消费所有队列:

```
    php artisan queue:listen
```

7. 失败任务存在`failed_tabke`

```
    php artisan queue:failed-table
```

# 使用队列步骤

## 「创建」任务类

```
    php artisan make:job SendNoticeEmail
```

> 原文中有:`--queued`  参数表示运行具有队列属性，即该任务类必须实现 ShouldQueue 接口。
> 自成Laravel 5.2版本后,不需要`--queued`


创建后，根据业务需求编写任务类，通常是某个模块所需的耗时功能。比如：发邮件、截图、生成压缩包、七牛云上传，等。

本文写的是发邮件, 在那之前需要一些邮件配置, 这里我用的是163邮箱:

```
    MAIL_DRIVER=smtp
    MAIL_HOST=smtp.163.com
    MAIL_PORT=25
    MAIL_FROM_ADDRESS=你的邮箱
    MAIL_FROM_NAME=LC
    MAIL_USERNAME=你的邮箱
    MAIL_PASSWORD=邮箱授权码
    MAIL_ENCRYPTION=null
```

- `app/Http/Jobs/SendNoticeEmail.php`

```php
    <?php
    
    namespace App\Jobs;
    
    use App\User;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;
    use Illuminate\Support\Facades\Mail;
    
    class SendNoticeEmail implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
        protected $user;
    
        /**
         * Create a new job instance.
         * 创建一个新的任务实例
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }
    
        /**
         * Execute the job.
         * 运行任务
         *
         * @return void
         */
        public function handle()
        {
            $user = $this->user;
    
            Mail::send('emails.notice', ['user' => $user], function ($msg) use ($user) {
                // 发件人的邮箱和用户名
                $msg->from('code0807@163.com', 'LC');
                // 收件人的邮箱地址,用户名,提示消息
                $msg->to($user->email, $user->name);
                $msg->subject('队列发送邮件');
            });
        }
    }
```

> [Mail](https://laravel.com/docs/5.5/mail)

## 「派发」任务

在某个业务逻辑处派发具体的任务，即「任务入队」：`$this->dispatch($job)`

```php
    <?php
    
    namespace App\Http\Controllers;
    
    use App\User;
    use App\Jobs\SendNoticeEmail;
    
    class UserController extends Controller
    {
        public function sendNoticeEmail()
        {
            $user = User::findOrFail(1);
    
            $job  = (new SendNoticeEmail($user))->onQueue('emails');
            //$job  = (new SendNoticeEmail($user))->delay('10');
    
            $this->dispatch($job);
    
            return view('emails.sendmail');
        }
    }
```

也支持使用 `$this->dispatchFrom()` 从请求中派发任务。

**只要使用了 `DispatchesJobs` trait 的类都可以直接通过 `$this->dispatch($job) `来派发任务到队列上。**

在将任务入队的时候，可以用 `onQueue(QUEUE_NAME)` 指定任务所属的队列；也可以用 `delay(secs)` 来延迟多少秒后运行。

## 处理「回调」

所注册的队列任务，运行完成后会被执行的回调操作，比如：数据库状态更新、邮件通知，等。主要有两种：

**完成通过**：`Queue::after($connection, $job, $data)`。

**失败通过**：`Queue::failing($connection, $job, $data)`。

通常在 `AppServiceProvider` 类的 `boot()` 方法中处理队列回调，其中失败事件可以直接在任务类中重写 `failed()` 方法来处理任务运行失败后的操作。

这里的 `$connection` 指的是队列连接类型，比如 `database`。

`$job` 是指当前任务实例本身的完整信息，其对应的类定义是：`Illuminate\Queue\Jobs\DatabaseJob`，其中有很多方法可以获取当前队列和任务对象的信息。

`$data` 是被序列化的 $job 对象属性对象，被存储在任务表的 `payload` 字段，可以通过 `unserialize($data['data']['command'])` 来恢复该实例对象。

## 「侦听」任务

代码逻辑写好之后，只是单纯将任务入队操作，并不会执行。需要启动 Laravel 侦听器来处理这些队列任务：

```
    php artisan queue:listen --queue=default,capture
```

`--queue=default,capture` 中代表此侦听器将处理哪些队列，排越靠左的队列，优先级越高。


## `queue:listen` 和 `queue:work` 区别 ？

> 本地调试的时候要使用 `queue:listen`，因为 `queue:work`在启动后，代码修改，`queue:work`不会再 `Load` 上下文，但是 `queue:listen`仍然会重新 `Load` 新代码。
>
> 其余情况全部使用 `queue:work` 吧，因为效率更高。

## 开发环境注意事项 ？

- 方便调试队列任务，可以将 `Queue Driver` 设置为 `sync`，将队列变成同步执行。
- Job 类中的 `handle` 方法是可以使用依赖注入的。

# 队列的释放和删除

从队列中释放一个任务，只是把这个任务重新放回队列，并不是从队列中删除。

要删除一个队列，需要删除队列中相应的记录，比如是数据库驱动的队列，就需要删除任务表中相关的记录。

也可以用 Artisan 命令来删除失败任务：

```
    php artisan queue:forget 5    # 删除掉某个失败任务
    
    php artisan queue:flush       # 删除所有失败任务
```

**非失败任务不能删除，因为任务成功后会自动删除。**

# 队列线上实践

在实际应用（线上）中，往往不是使用 `queue:listen` 来处理队列任务，而是通过 `queue:work` 配合 `--daemon` 选项，再加上 <u>[supervisor](http://supervisord.org/configuration.html)</u> 进程管理器，来处理线上环境的队列任务。

相关配置及脚本如下：

- **队列任务批量部署脚本**

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

这里部署的后台处理进程个数根据服务器配置情况而定，这里是在 `4 核／16G／20M` 的阿里云 ECS 机器上部署的。

- 安装&使用 `supervisor`

```
    # 以 CentOS 6 为例
    
    yum info supervisor
    yum install -y supervisor
    
    service supervisord start|stop|restart
    
    supervisorctl reread
    supervisorctl start proj-worker:*
    
    vim /etc/supervisord.conf
```

- `supervisor` 对应配置

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

注意这里的 `numprocs` 数最好和 `deploy.sh` 里面的 `workers` 数量保持一致，因为确保每个后台运行的队列处理进程都能被 `supervisor` 监控到。

其中，`%(process_num)2d` 对应的是数 `numprocs` 内的序号。`0` 表示不足多少位（这里是2位）时填充。


# 参考

- [Laravel 的队列系统介绍](https://laravel-china.org/docs/laravel/5.5/queues/1324)
- [Laravel队列小结](https://segmentfault.com/a/1190000012384121)
- [Laravel 队列系列 —— 基于 Redis 实现任务队列的基本配置和使用](http://laravelacademy.org/post/2012.html)