---
title: Linux 命令 「crontab」
date: 2018-06-07 13:42:57
categories: Linux cmd
tags: [Linux]
---

# `crontab`

**crontab命令**被用来提交和管理用户的需要周期性执行的任务，与windows下的计划任务类似

<!-- more -->

# 命令参数

- `-e`: 编辑该用户的计时器设置
- `-l`: 列出该用户的计时器设置
- `-r`: 删除该用户的计时器设置
- `-u`<用户名称>: 指定要设定计时器的用户名称

**Linux下的任务调度分为两类：系统任务调度和用户任务调度。**

# 系统任务调度

系统周期性所要执行的工作，比如写缓存数据到硬盘、日志清理等。
在/etc目录下有一个crontab文件，这个就是系统任务调度的配置文件。

`/etc/crontab`文件内容:

```crontab
    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root
    HOME=/
    
    # For details see man 4 crontabs
    
    # Example of job definition:
    # .---------------- minute (0 - 59)
    # |  .------------- hour (0 - 23)
    # |  |  .---------- day of month (1 - 31)
    # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name  command to be executed
```

> **记忆口令: 分 时 日 月 周 cmd**

## 参数解读

- 第一行SHELL变量指定了系统要使用哪个`shell`，这里是`bash`.
- 第二行`PATH`变量指定了系统执行命令的路径
- 第三行`MAILTO`变量指定了`crond`的任务执行信息将通过电子邮件发送给`root`用户，
  如果`MAILTO`变量的值为空，则表示不发送任务执行信息给用户
- 第四行的`HOME`变量指定了在执行命令或者脚本时使用的主目录(可以省略)

## 特殊字符

- 星号(`*`): 代表所有可能的值.
  - 例如: month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
  
- 逗号(`,`): 可以用逗号隔开的值指定一个列表范围,
  - 例如: "1,2,4,5,6"
  
- 中杠(`-`): 可以用整数之间的中杠表示一个整数范围,
  - 例如 "2-6"表示"2,3,4,5,6"
  
- 正斜线(`/`): 可以用正斜线指定时间的间隔频率, 
  - 例如: “0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，
  - 例如: */10，如果用在minute字段，表示每十分钟执行一次。


# 用户任务调度

所有用户定义的`crontab`文件都被保存在`/var/spool/cron`目录中。
其文件名与用户名一致，使用者权限文件如下：

```crontab
    /etc/cron.deny      该文件中所列用户不允许使用crontab命令
    /etc/cron.allow     该文件中所列用户允许使用crontab命令
    /var/spool/cron/    所有用户crontab文件存放的目录,以用户名命名
```


# `crond`服务

- `/sbin/service crond start` 启动服务
- `/sbin/service crond stop` 关闭服务
- `/sbin/service crond resatrt` 重启服务
- `/sbin/service crond reload` 重新载入配置

- `service crond status` 查看`crontab`服务状态
- `service crond start` 手动启动`crontab`服务
- `ntsysv`  查看`crontab`服务是否设置为开机启动
- `chkconfig -level 35 crond on` 加入开机自动启动


# 实例

- 每一分钟执行一次 (固定间隔)

```$xslt
    * * * * * command
```

- 每晚的21:30重启smb (指定时间)

```$xslt
    30 21 * * * /etc/init.d/smb restart
```

- 每天18 : 00至23 : 00之间每隔30分钟重启smb (时间区域内固定间隔)

```$xslt
    0,30 18-23 * * * /etc/init.d/smb restart
```