---
title: Linux 命令 「chage」
date: 2018-07-09 09:22:20
categories: Linux cmd
tags: [Linux]
---

> **chage命令**是用来修改帐号和密码的有效期限。

<!-- more -->

# 语法

```
    chage [选项] 用户名
```

# 选项

- `-m`:   密码可更改的最小天数, 为零时代表任何时候都可以更改密码
- `-M`:   密码可保持有效的最大天数
- `-W`:   用户密码到期前, 提前收到警告信息的天数
- `-E`:   账号到期的日期, 过了这天, 此账号将不可用
- `-d`:   上一次更改的日期
- `-i`:   停滞时期, 如果一个密码已过期这些天,那么此账号将不可用
- `-l`:   列出当前的设置. 由非特权用户确定他们的密码或账号何时过期

# 实例

- 可以编辑 `/etc/login.defs` 来设定几个参数，以后设置口令默认就按照参数设定为准:

```
     PASS_MAX_DAYS  99999
     PASS_MIN_DAYS  0
     PASS_MIN_LEN   5
     PASS_WARN_AGE  7
```

- 在`/etc/default/useradd`可以找到如下2个参数进行设置:

```
    [root@izj6c6djex81rijczh0t8yz default]# cat useradd 
    # useradd defaults file
    GROUP=100
    HOME=/home
    INACTIVE=-1
    EXPIRE=
    SHELL=/bin/bash
    SKEL=/etc/skel
    CREATE_MAIL_SPOOL=yes
```

通过修改配置文件，能对之后新建用户起作用，而目前系统已经存在的用户，则直接用`chage`来配置。

- 我的服务器root帐户密码策略信息如下:

```
    [root@izj6c6djex81rijczh0t8yz default]# chage -l root
    Last password change					: Jan 02, 2018  # 最近一次密码修改时间
    Password expires					: never  # 密码过期时间
    Password inactive					: never  # 密码失效时间
    Account expires						: never  # 账户过期时间
    Minimum number of days between password change		: 0     # 两次改变密码之间相距的最小天数
    Maximum number of days between password change		: 99999 # 两次改变密码之间相距的最大天数
    Number of days of warning before password expires	: 7     # 在密码过期之前警告的天数
```

- 我可以通过如下命令修改我的密码过期时间:

```
    [root@izj6c6djex81rijczh0t8yz default]# chage -l root
    Last password change					: Jan 02, 2018
    Password expires					: Sep 27, 2020
    Password inactive					: never
    Account expires						: never
    Minimum number of days between password change		: 0
    Maximum number of days between password change		: 999
    Number of days of warning before password expires	: 7
```

- 然后通过如下命令设置密码失效时间:

```
    [root@izj6c6djex81rijczh0t8yz default]# chage -I 999 root
    [root@izj6c6djex81rijczh0t8yz default]# chage -l root
    Last password change					: Jan 02, 2018
    Password expires					: Sep 27, 2020
    Password inactive					: Jun 23, 2023
    Account expires						: never
    Minimum number of days between password change		: 0
    Maximum number of days between password change		: 999
    Number of days of warning before password expires	: 7
```