---
title: Git 服务器 搭建/自动部署/远程连接
date: 2018-06-14 09:41:20
categories: Git
tags: [Git]
---

> 对于不想给Github之类的代码托管掏钱的人.搭建一个自己的Git服务器很有必要

<!-- more -->

# `GIT`服务器搭建

- 1. 安装`git`

```
    yum install git
    
    // Ubuntu
    sudo apt-get install git
```

- 2. 创建`git`用户

```
    git useradd git
```

> 如果忘记git用户密码: `sudo passwd git` 重置即可

- 3. 创建`git`裸仓库

```
    sudo git init --bare sample.git
```

- 4. 创建证书登录

收集所有需要登录的用户的公钥，就是他们自己的`id_rsa.pub`文件，
把所有公钥导入到`/home/git/.ssh/authorized_keys`文件里，一行一个。

```
    git config --global user.email "code0809@163.com"
    git config --global user.name "SoulMate94."
    
    ssh-keygen -t rsa -C "code0809@163.com"
```

- 5. 初始化`Git`仓库

```
    sudo git init --bare dirname.git        // dirname 设置为自己想要的项目目录名称
```

`Git`就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的`Git`仓库纯粹是为了共享，
所以不让用户直接登录到服务器上去改工作区，并且服务器上的`Git`仓库通常都以`.git`结尾。然后，把`owner`改为`git`：

```
    sudo chown -R git:git dirname.git
```

- 6. 禁用shell登录：

出于安全考虑，第二步创建的`git`用户不允许登录`shell`，这可以通过编辑`/etc/passwd`文件完成。找到类似下面的一行：

```
    vim /etc/passwd
    git:x:1002:1002:,,,:/home/git:/bin/bash
```

改为:

```
    git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

> 这样，`git`用户可以正常通过`ssh`使用`git`，但无法登录`shell`，
因为我们为`git`用户指定的`git-shell`每次一登录就自动退出。


- 7. 克隆远程仓库：

```
    git clone git@server:/dirname.git
    
    Cloning into 'dirname'...
    warning: You appear to have cloned an empty repository.
```

# 添加远程仓库到本地

```
    git init
    git remote add origin git@server:/dirname.git
```

## 查看远程仓库

```
    git remote -v
    
    例:
    origin  git@47.91.221.85:/home/wwwroot/git/sample.git (fetch)
    origin  git@47.91.221.85:/home/wwwroot/git/sample.git (push)
```

## 操作

```
    // 在第一次进行push时,我们加上-u参数,后期push时就不用再加-u参数
    git push -u origin master
    
    // 正常工作中
    git pull origin master      // 拉取仓库代码, 解决冲突
    git add .                   // 将文件添加到本地版本库
    git commit -m "注释"         // 将文件修改提交到仓库
    git push origin master      // 这里origin 仅代表远程仓库在本地的连接名称
```


## 远程仓库操作命令:

- 检出

```
    git clone git@server:/dirname.git
```

- 查看

```
    git remote -v
```

- 添加

```
    git remote add remoteName(连接名) git@server:/dirname.git
```

- 删除

```
    git remote rm remoteName(连接名)
```

- 修改

```
    git remote set-url --push remoteName(连接名) git@server:/dirname.git
```

- 拉取

```
    git pull remoteName(连接名)
```

**Git pull 强制覆盖本地文件 ?**

```
    git fetch --all
    git reset --hard origin/master
    git pull
```

- 推送

```
    git push remoteNmae(连接名)
```

# `PHPStorm`连接远程服务器

- 1. 首先打开`phpstorm`选择：`Tools->Deployment->Configuration`

![连接远程服务器](https://caoxl.com/images/deploy1.png)

- 2. 点击新建

![配置远程服务器](https://caoxl.com/images/deploy2.png)

- 3. 操作测试/成功截图

![上传成功](https://caoxl.com/images/deploy3.png)


# 远程服务器自动部署

> `sample` 为自定义目录名称

- `sample.git/hooks/post-receive`

```
    #!/bin/sh
    unset GIT_INDEX_FILE
    echo "=> update source code to the latest"
    while read oldrev newrev ref
    do
        branch=$(git rev-parse --symbolic --abbrev-ref $refname)
        echo "$branch"
        git --work-tree=/home/wwwroot/sample --git-dir=/home/wwwroot/git/sample.git
        git checkout $branch -f
    done
```

**需要自定义:**

> `--work-tree=/home/wwwroot/sample` 是工作目录
> `--git-dir=/home/wwwroot/git/sample.git` 是`.git`目录

- `sample.git/config`

```
    [core]
            repositoryformatversion = 0
            filemode = true
            bare = false
            logallrefupdates = true
            ignorecase = true
            precomposeunicode = true
            worktree = /home/wwwroot/sample
    
    [receive]
            denycurrentbranch = ignore
```

**需要自定义:**

> `worktree = /home/wwwroot/sample` 是工作目录

## 注意

***配置里面不能有`//`注释符!***