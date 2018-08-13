---
title: Laravel/Lumen 安装器
date: 2018-03-01 15:00:27
categories: Laravel
tags: [Laravel, Lumen]
---

`Laravel` and `Lumen` 快捷安装

<!-- more -->

# Laravel

```
    composer global require "laravel/installer=~1.1"

    laravel new sites
```


# Lumen

```
    composer global require "laravel/lumen-installer"

    # or
    composer require laravel/lumen-installer

    lumen new sites
```

# FAQ

- sh: command not found laravel or lumen 

```
    export PATH="~/.composer/vendor/bin:$PATH" 
```

vim ~/.zshrc

```
    alias laravel='~/.composer/vendor/bin/laravel' 

    source ~/.zshrc
```

关闭终端输入 `laravel new sites`

