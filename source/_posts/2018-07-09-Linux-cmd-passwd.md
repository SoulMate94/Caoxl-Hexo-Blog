---
title: Linux 命令 「passwd」
date: 2018-07-09 09:53:25
categories: Linux cmd
tags: [Linux]
---

> **passwd命令**用于设置用户的认证信息，包括用户密码、密码过期时间等。
系统管理者则能用它管理系统用户的密码。只有管理者可以指定用户名称，一般用户只能变更自己的密码。

<!-- more -->

# 语法

```
    passwd (选项) (参数)
```

# 选项

- `-d`:   删除密码, 仅有系统管理者才能使用
- `-f`:   强制执行
- `-k`:   设置只有在密码过期失效后, 方能更新
- `-l`:   锁住密码
- `-s`:   列出密码的相关信息,仅有系统管理者才能使用
- `-u`:   解开已上锁的账号

# 参数

- 用户名: 需要设置密码的用户名

# 扩展

**与用户、组账户信息相关的文件:**

- 存放用户信息:

```
    /etc/passwd
    /etc/shadow
```

- 存放组信息

```
    /etc/group
    /etc/gshadow
```

- 用户信息文件分析 (每项用 `:` 隔开)

```
    例如: git:x:1002:1002::/home/git:/usr/bin/git-shell
    git     // 用户名
    x       // 口令、密码
    1002    // 用户id (0代表root、普通新建用户从500开始)
    1002    // 所在组
    :       // 描述
    /home/git/  // 用户主目录
    /usr/bin/git-shell  // 用户缺省Shell
```

# 实例

- 如果是普通用户执行`passwd`只能修改自己的密码。如果新建用户后，要为新用户创建密码，则用`passwd`用户名，注意要以root用户的权限来创建。

```
    [root@localhost ~]# passwd linuxde    //更改或创建linuxde用户的密码；
    Changing password for user linuxde.
    New UNIX password:          //请输入新密码；
    Retype new UNIX password:   //再输入一次；
    passwd: all authentication tokens updated successfully. //成功；
```

- 普通用户如果想更改自己的密码，直接运行`passwd`即可，比如当前操作的用户是`linuxde`。

```
    [linuxde@localhost ~]$ passwd
    Changing password for user linuxde. //更改linuxde用户的密码；
    (current) UNIX password:   //请输入当前密码；
    New UNIX password:         //请输入新密码；
    Retype new UNIX password:  //确认新密码；
    passwd: all authentication tokens updated successfully. //更改成功；
```

- 让某个用户不能修改密码，可以用`-l`选项来锁定:

```
    [root@localhost ~]# passwd -l linuxde    //锁定用户linuxde不能更改密码
    Locking password for user linuxde.
    passwd: Success           //锁定成功
    
    [linuxde@localhost ~]# su linuxde   //通过su切换到linuxde用户
    [linuxde@localhost ~]$ passwd      //linuxde来更改密码
    Changing password for user linuxde.
    Changing password for linuxde
    (current) UNIX password:          //输入linuxde的当前密码
    passwd: Authentication token manipulation error     //失败，不能更改密码
```

- 再来一例:

```
    [root@localhost ~]# passwd -d linuxde  //清除linuxde用户密码
    Removing password for user linuxde.
    passwd: Success                         //清除成功
    
    [root@localhost ~]# passwd -S linuxde    //查询linuxde用户密码状态
    Empty password.                         //空密码，也就是没有密码
```

**注意：当我们清除一个用户的密码时，登录时就无需密码，这一点要加以注意。**

```
    [root@izj6c6djex81rijczh0t8yz ~]# su caoxl    # 无需密码
    [caoxl@izj6c6djex81rijczh0t8yz root]$ su lc   # 需要密码
    Password:
```