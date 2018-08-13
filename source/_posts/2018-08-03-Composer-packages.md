---
title: Composer 开发自己的组件
date: 2018-08-03 14:09:35
categories: DevTool
tags: [Composer, Packagist]
---

> 知识没有你的我的, 只有学习顺序的先后, 你从别人那学,我从你这学, 别人又从我这里学,多好~~。
> > 写个 `Composer` 组件玩玩，顺便总结下 `Composer` 的使用。

<!-- more -->

# 基础知识

## 组件化

现代 Web 开发已经不再流行开发一个巨型框架了，反过来，我们更关心的是通过甄选一个个由专业人士编写的、可以**具有互操作特性的组件**，来组装解决方案。

> 互操作性：组件之间可以很方便的互相利用。

`PHP Web` 开发也不例外，`composer` 推动了 `PHP` 生态的组件化进程。

> 当我们准备创建 PHP 项目的时候，优先考虑的不是使用什么全能型框架而是考虑能否通过少量组件就可以满足需求。
> > 此外，一些大型框架也是以组件的方式被 `composer` 管理。

## `autoloader`

当通过 `composer` 下载 `PHP` 组件的同时，`composer` 将会扫描每个组件的 `composer.json` 文件，并为我们的整个项目生成好了一份兼容 `PSR` 规范的 `autoloader`，位于 `vendor/autoload.php`，项目通过该引入自动加载器，便能够在项目范围内使用各个组件提供的功能。


## `composer.json` 部分字段解释

### `require`

列出了该 PHP 项目/组件的所有依赖，包括 PHP 版本以及其他库。该字段中的依赖包将会在开发环境和生产环境中被安装

### `require-dev`

和 `required` 作用大体一致，不过只是列举出开发当前项目/组件期间需要用的依赖，比如单元测试框架 `phpunit` 等。

> 生产环境中，该列表中的依赖不应该被安装。

### `suggest`

`composer` 不会安装这里指出的组件，**仅仅起到提示作用**：提示这些组件可能会对当前项目/组件有用。


### `autoload`

该字段用于指示 `composer autoloader` 如何进行自动加载。

一般来说，目前我们应该遵守并使用 `psr-4` 规范。举例说明：

```
    "autoload": {
        "psr-4": {
            "A\\B\\C": "src/"
        }
    }
```

该例子的意思是，当需要找到命名空间为 `A\B\C` 的类文件时，到当前 `composer.json` 所在的路径(也是项目的根路径)下面的 `src/` 目录下以此为跟路径开始找。

**其中，两个反斜线 `\\` 不能写漏。**

### 举例: `composer.json`

```
    {
        "name": "laravel/lumen",
        "description": "The Laravel Lumen Framework.",
        "keywords": ["framework", "laravel", "lumen"],
        "license": "MIT",
        "type": "project",
        "require": {
            "php": "^7.0",
            "laravel/lumen-framework": "5.4.*",
            "vlucas/phpdotenv": "~2.2",
            "firebase/php-jwt": "^5.0",
            "symfony/var-dumper": "^3.3",
            "predis/predis": "^1.1",
            "illuminate/redis": "^5.4",
            "qiniu/php-sdk": "^7.1",
            "maatwebsite/excel": "~2.1.0",
            "jpush/jpush": "^3.5",
        },
        "require-dev": {
            "fzaninotto/faker": "~1.4",
            "phpunit/phpunit": "~5.0",
            "mockery/mockery": "~0.9"
        },
        "autoload": {
            "psr-4": {
                "App\\": "app/"
            },
            "files": [
                "app/functions.php"
            ]
        },
        "autoload-dev": {
            "classmap": [
                "tests/",
                "database/"
            ]
        },
        "scripts": {
            "post-root-package-install": [
                "php -r \"copy('.env.example', '.env');\""
            ]
        },
        "minimum-stability": "dev",
        "prefer-stable": true,
        "repositories": {
            "packagist": {
                "type": "composer",
                "url": "https://packagist.phpcomposer.com"
            }
        }
    }
```

# 搭建私有 `composer` 仓库

> [Handling private packages](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md#setup)

```
    # Via composer/source
    composer create-project composer/statis --stability=dev --keep=vcs
    php bin/satic build satic.json /path/to/output-web-dir/ -vvv
```

**说明:** `/path/to/output-web-dir/`  此处为自定义目录

# `Composer`组件开发流程

## 初始化

`composer` 项目是由一定规范限制的，可以通过如下命令初始化一个 `composer` 组件。

```
    mkdir test && cd tests
    composer init
```

出现以下内容:

```
    $ composer init
    
    
      Welcome to the Composer config generator
    
    
    This command will guide you through creating your composer.json config.
    
    Package name (<vendor>/<name>) [97321/test]: caoxl/caoxl
    Description []: caoxl_package
    Author [SoulMate94. <code0809@163.com>, n to skip]:
    Minimum Stability []: dev
    Package Type (e.g. library, project, metapackage, composer-plugin) []:
    License []: MIT
    
    Define your dependencies.
    
    Would you like to define your dependencies (require) interactively [yes]?
    Search for a package:
    Would you like to define your dev dependencies (require-dev) interactively [yes]?
```

结束时会生成一份 `composer.json` 文件。当然也可以直接创建 `composer.json` 然后定义同样的属性。

- 就像下面这样:

```
    {
        "name": "vendor_name/component_name",
        "description": "desc",
        "type": "library",
        "license": "MIT",
        "authors": [
            {
                "name": "your_name",
                "email": "your@email.com"
            }
        ],
        "minimum-stability": "dev",
        "require": {
            "php": ">=5.4.0"
        },
        "autoload": {
            "psr-4": {
                "<NAMESPACE>\\": "src/"
            }
        }
    }
```

## 开发

接下来就是敲代码了。

注意，为了规范和统一，一般在 `src` 目录中放主要的源码，然后 `composer.json` 使用的命名空间和这个 `src` 关联。

- 示例: 新建`/src/test.php`

```
    <?php
    
    // 获取扩展名
    function get_file_ext($file){
        return strtolower(pathinfo($file,PATHINFO_EXTENSION));
    }
```

## 提交到 `GitHhub`

代码写完了, 提交到`Github`托管

- 在`Github`上新建一个项目:

```
    echo "# composer_vendor" >> README.md
    git init
    git add README.md
    git commit -m "first commit"
    git remote add origin git@github.com:SoulMate94/composer_vendor.git
    git push -u origin master
```

> 至此代码已经提交到`github`托管; [SoulMate94/composer_vendor](https://github.com/SoulMate94/composer_vendor)

## 提交到 `Packagist`

注册 `packgist` 账号后，点击 `Submit`，输入 仓库地址，点击 `Check`，检查无误确认提交。

![Packagist-Submit](https://caoxl.com/images/packagist-submit.png)

> 成功后可以看到 <u>[Packagist-caoxl/caoxl](https://packagist.org/packages/caoxl/caoxl)</u>

## 设置自动更新

在 `GitHub` 仓库`Settings`中找到 `Services`，搜索 `Packagist`，然后填好：

- 用户名: `Packagist` 用户名
- `Token`: 在 `Packagist`个人设置中可以查看
- 域名: `packagist.org` -- 此处填`https://packagist.org`即可

![Github-packagist](https://caoxl.com/images/github-packagist.png)

## 测试

如果上面测试无误则可以在任何地方使用 `composer require <Vendor>/<Package>` 来下载并使用你开放的 `composer` 组件了。

```
    composer require caoxl/caoxl
```

# FAQ

## `composer.lock` 是否应该纳入版本控制？

> **是, 这是`lock`存在的意义**

版本库中的 `composer.lock` 确保了协作开发和在任何环境安装组件的时候都是同一个版本号

此外，**尽量不要在正式环境执行 `composer update`**。当某个开发者要更新某个组件的时候，应该现在他的环境更新该组件，测试后在提交 `composer.lock` 到版本库，其他开发者在不需要更新任何组件的情况下只需要执行 `composer install` 即可。

**生产环境只允许执行 `composer install`。**

## 对文件名／目录名大小写敏感导致 “Class Not found”？

```
    "autoload": {
        "classmap": ["folder1/", "folder2/"]
    }
```

> [How to add case insensitive autoloading using composer generated classmap?](https://stackoverflow.com/questions/20235922/how-to-add-case-insensitive-autoloading-using-composer-generated-classmap)

## 缓存导致找不到`PHP`类 ?

```
    composer dump-autoload
```

# 参考

- [Packagist - The PHP Package Repository](https://packagist.org/) && [Packagist/Composer中国全量镜像](https://pkg.phpcomposer.com/)
- [composer-doc](https://getcomposer.org/doc/)
- [Creating your first Composer/Packagist package](https://blog.jgrossi.com/2013/creating-your-first-composer-packagist-package/)
- [composer/satis](https://github.com/composer/satis)
- [使用 satis 搭建私有的 Composer 包仓库](http://www.ps-aux.com/%E4%BD%BF%E7%94%A8%20satis%20%E6%90%AD%E5%BB%BA%E7%A7%81%E6%9C%89%E7%9A%84%20Composer%20%E5%8C%85%E4%BB%93%E5%BA%93.html)
- [Composer: It’s All About the Lock File](https://www.engineyard.com/blog/composer-its-all-about-the-lock-file)
- [Should composer.lock be committed to version control?](https://stackoverflow.com/questions/12896780/should-composer-lock-be-committed-to-version-control)
- [Composer 使用日志](http://blog.cjli.info/2017/07/22/Composer-Notes/)