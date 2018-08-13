---
title: Linux 自定义命令
date: 2018-07-02 10:27:14
categories: Linux
tags: [Linux, 自定义命令]
---

为了节约时间（偷懒），下面介绍一下`mac`/`Linux`下使用`alias`配置常用命令。

<!-- more -->

# 自定义命令

- `Windows`下编辑`~/.bash_profile`

- `Linux`下编辑`~/.bashrc`

# 直接上我的配置

```
    # Alias Cmd
    
    # PHP
    alias serve='php artisan serve';
    
    # Git
    alias ls='ls --color=auto';
    alias ll='ls -lh';
    alias ld='ll |grep "^d"';
    alias psw='ps -a --windows';
    
    alias gst='git status';
    alias gad='git add -A';
    alias gcm='git commit -m $1';
    alias gcb='git checkout $1';
    alias gpushb="git push origin $1";
    alias gpushm="git push origim master";
    alias gpullb="git pull origin $1";
    alias gpullm="git pull origin master";
    alias gcb="git checkout $1";
    alias gcc="git checkout caoxl";
    alias gm="git checkout master";
    alias gb="git branch";
    alias gba="git branch -a";
    alias gmb="git merge $1";
    alias gmm="git merger master";
    alias gdf="git diff";
    alias grv="git remote -v";
    alias gll="git log --graph --abbrev-commit --decorate --all --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(dim white) - %an%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n %C(white)%s%C(reset)'";
    
    # github
    
    alias github='git config --global user.name "SoulMate94" && git config --global user.email "code0809@163.com" && ssh-keygen -t rsa -C "code0809@163.com"';
    
    # FILE
    alias ..='cd ..';
    alias ....='cd ../../';
    
    
    # VI/VIM
    alias vi='vim';
    
    
    # .bashrc (Windows下为`.bash_profile`)
    alias sss='source ~/.bashrc';
    alias vbash='vim ~/.bashrc';
    
    
    # Nginx
    alias nginx.start='service nginx start';
    alias nginx.stop='nginx -s stop'; // 强制关闭
    alias nginx.process='ps -ef | grep nginx';
    alias nginx.reload='nginx -s reload';
    alias nginx.kill='pkill -9 nginx';
    alias nginx.restart='service nginx restart';
    
    
    # Artisan
    alias autoload='composer dump-autoload';
    alias art.key='php artisan key:generate';
    alias migrate='php artisan migrate';
    alias route.list='php artisan route:list';
    alias seed='php artisan db:seed';
    alias key='php artisan key:generate';
    
    # Laravel-admin
    alias admin.install='composer require encore/laravel-admin "1.5.*"';
    alias admin.install.2='php artisan vendor:publish --provider="Encore\Admin\AdminServiceProvider" && php artisan admin:install';
    alias admin.helper='composer require laravel-admin-ext/helpers && php artisan admin:import helpers';
```

