---
title: PHP 「安装日志」
date: 2018-08-21 15:29:39
categories: PHP
tags: [PHP, Install, Linux]
---

> 总结一些和PHP有关的 「 安装 」

<!--more-->

# PECL

> PECL 的全称是 The PHP Extension Community Library ，是一个开放的并通过 PEAR(PHP Extension and Application Repository，PHP 扩展和应用仓库)打包格式来打包安装的 PHP扩展库仓库。通过 PEAR 的 Package Manager 的安装管理方式，可以对 PECL 模块进行下载和安装。

<u>[PHP Extension Community Library](https://pecl.php.net/)</u>。提供了一个 PHP 所有已知扩展的下载和托管目录，比如 <u>[fileinfo](https://pecl.php.net/package/fileinfo)</u>, <u>[APC](https://pecl.php.net/package/APC)</u>, <u>[Xdebug](https://pecl.php.net/package/Xdebug)</u> 等，采用 C 语言编写。

# PEAR

<u>[PHP Extension and Application Repository](https://pear.php.net/)</u>。简单说有 **3** 个作用：

- 打包和安装 C 语言形式的 PECL 扩展
- 托管 PHP 代码形式的应用仓库、提供可复用的组件和依赖管理（虽然做的不好）
- 作为 PHP 编码规范（曾经的）

## PEAR vs PECL

[官方的](https://secure.php.net/manual/zh/install.pecl.intro.php) 解释是：PECL 是通过 PEAR 打包系统来的 PHP 扩展库仓库。

> PECL代表PHP扩展社区库，它具有用C编写的扩展，可以加载到PHP中以提供其他功能。您需要具有管理员权限，C编译器和相关工具链才能安装这些扩展。
>  
> PEAR是PHP扩展和应用程序库，它具有用PHP编写的库和代码。您只需下载，安装并包含在您的代码中即可。
  
## 安装 PEAR／PECL

以在 `macOS` 下为例简述流程：

```
    # Download pear
    curl -O http://pear.php.net/go-pear.phar
    
    # Install pear/pecl
    sudo php -d zend_detect_unicode=0 go-pear.phar
```

然后选择安装的路径，一般要手动设置安装路径（1）为 `/usr/local/pear`，和二进制文件路径（4）为 `/usr/local/bin`。设置好后输入all 或者按回车即可开始安装。

如果没出意外，就可以检测是否安装成功：

```
    pear version
    pecl version
```

最后，如果要使用任何由 pear 下载的扩展或者应用，必须在 `php.ini` 中更新 `include_path` 路径：

```
    sudo vim /etc/php.ini
```

找到 `Paths and Directories` 在下面添加：

```
    include_path="/usr/local/pear/share/pear"
```

这个 `include_path` 的值，是安装成功后提示你的路径。

然后重启：`sudo apachectl restart`。


## 通过 PECL 和 PEAR 安装 PHP 扩展

最后就可以使用 pear 和 pecl 安装 PHP 扩展了。比如安装 `vld` 扩展：

```
    sudo pecl install vld-0.14.0    # https://pecl.php.net/package/vld
```

注意：这里的扩展名必须是在 `pecl.php.net` 上面能搜到的准确扩展名称及其版本号。

如果没报错，则在 `php.ini` 中引入刚刚编译成功的 `vld.so`，重启 apache 即可。

### 说明

通过 pecl 或者 pear 安装 PHP 扩展的时候，**必须确保 perl 或者 pear 和 PHP 的版本一致**，否则安装不会成功。

# Composer

Composer 是现代 PHP（>5.3） 中一个优秀的包管理器，通常和 <u>[Packagist](https://packagist.org/)</u> 一起工作，可以很方便地管理大量的 PHP 代码依赖。

## Composer 与 Packagist

Composer 中管理的依赖都写在 `composer.json` 中，而最终安装的时候，都是去 `Packagist` 下载。

Packagist 是一个官方的 Composer 兼容包仓库，但是并不是唯一的，比如国内还有一个 <u>[Packagist 全量镜像](https://pkg.phpcomposer.com/)</u>。Composer 获取软件包的地址，是可以修改的，比如替换默认源就可以：

```
    # 全局
    composer config -g repo.packagist composer https://packagist.phpcomposer.com
    # 单个项目
    composer config repo.packagist composer https://packagist.phpcomposer.com
```

## Composer vs PEAR

PHP 有很多可供使用的库、框架和组件。通常你的项目都会使用到其中的若干项，这些就是项目的依赖。

对于现代 PHP 来说，主要由 Composer 管理 PHP 依赖，而在以前，则是 PEAR 来管理的。两者的主要区别有：

- Composer 更活跃和成熟，PEAR 则将被弃用。
- Composer 更灵活，可以为单独某个项目管理依赖，也可全局安装，而 PEAR 只能全局安装。（好处是当多个项目共同使用同一个扩展包的同一个版本, 坏处是如果你需要使用不同版本的话，就会产生冲突）
- PEAR 需要扩展包有专属的结构，开发者在开发扩展包的时候要提前考虑为 PEAR 定制，否则后面将无法使用 PEAR，而 Composer 的使用规范对所有组件都是统一的。
- 通过 Composer + Packgist，能安装到的软包数量远远多于 PEAR。
- 在依赖管理方面，Composer 更专业。
- PEAR 能安装 PECL 扩展，而 Composer 不能。这算是当前 PEAR 唯一比得过 Composer 的地方了，不过 Composer 目前又有一个不算特别成熟的替代方案：
<u>[GitHub: FriendsOfPHP/pickle。](https://github.com/FriendsOfPHP/pickle)</u>

# 软件包管理器

## Linux

使用软件包管理器比较方便，不仅可以安装 PHP，也可以安装 PHP 扩展。缺点就是版本默认较低。

### yum/dnf

CentOS 使用 yum 安装的 PHP 版本很低，需要先开启 `EPCL` 和 `REMI` 源。

```
    wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
    wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
    rpm -Uvh remi-release-7*.rpm epel-release-7*.rpm
    
    vim /etc/yum.repos.d/remi.repo
```

添加如下内容后保存退出：

```
    [remi]
    name=Les RPM de remi pour Enterprise Linux 6 - $basearch
    #baseurl=http://rpms.famillecollet.com/enterprise/6/remi/$basearch/
    mirrorlist=http://rpms.famillecollet.com/enterprise/6/remi/mirror
    enabled=1
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
    
    [remi-php56]
    name=Les RPM de remi de PHP 5.6 pour Enterprise Linux 6 - $basearch
    #baseurl=http://rpms.famillecollet.com/enterprise/6/php56/$basearch/
    mirrorlist=http://rpms.famillecollet.com/enterprise/6/php56/mirror
    # WARNING: If you enable this repository, you must also enable "remi"
    enabled=1
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
```

然后就可以搜索指定的版本并安装了：

```
    yum search php | grep ^php
    yum install php php-gd php-mysql php-mcrypt
```

### apt-get

```
    echo "deb http://packages.dotdeb.org wheezy-php56 all" >> /etc/apt/sources.list.d/dotdeb.list
    echo "deb-src http://packages.dotdeb.org wheezy-php56 all" >> /etc/apt/sources.list.d/dotdeb.list
    wget http://www.dotdeb.org/dotdeb.gpg -O- | apt-key add - 
    apt-get update
    apt-get install php5-cli php5-fpm    # or whatever package you might need
```

## Mac

Mac 默认是没有包管理器的，但是有解决方案，比如<u>[Homebrew](https://brew.sh/)</u> 。

### Homebrew

```
    brew search php
    
    # 安装需要的版本
    brew install php56
    brew install php72
```

### Macports

```
    sudo port install php56
    sudo port install php72
    
    # Switch PHP Version
    sudo port select --set php php72
```

# 源码安装

源码安装可以定制自己的环境，也能在一定程度上提高性能，只是流程稍微繁琐（其实还好）。

## 源码安装 PHP

官方有很全的<u>[指导手册](https://secure.php.net/manual/en/install.php)</u>，下面只以在 Linux 下安装 php（work with nginx） 场景示例过程：

```
    # 1. Download（http://www.php.net/downloads.php）
    wget http://cn2.php.net/distributions/php-5.6.29.tar.bz2
    bzip2 -d php-5.6.29.tar.bz2
    
    # 2. Compile and install
    cd php-5.6.29
    ./configure --help
    ./configure --enable-fpm --with-mysql
    make
    sudo make install
    
    # 3. Configuration files ready
    cp php.ini-development /usr/local/php/php.ini
    cp /usr/local/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf
    cp sapi/fpm/php-fpm /usr/local/bin
    
    # 4. Necessary configs
    vim /usr/local/php/php.ini
    	cgi.fix_pathinfo=0
    
    vim /usr/local/etc/php-fpm.conf
        user = www-data
        group = www-data
        
    # 4. start php-fpm
    /usr/local/bin/php-fpm
```

## 源码安装 PHP 扩展

安装 PHP 扩展的前提：

- 已安装 PHP 本身
- 必要的基础依赖已经安装，比如 gcc／make
- 知道要安装的扩展的安装依赖（参考 PECL 上的说明）

### 源码安装官方扩展

Linux 下源码安装 PHP 官方（内置）扩展基本流程如下：

```
    # 1. Basic requirements
    yum install php-devel php-mysqlnd gcc libtool
    
    # 2. Extensions dependency
    # differ from ext to ext ...
    
    # 3. Have source code of current PHP version
    cd /path/to/php-x.y.z
    
    # 4. Assume we install fileinfo extension
    cd fileinfo
    phpize
    ./configure --enable-fileinfo
    make
    sudo make install
    
    # 5. Enable new in php.ini
    cd `php-config --extension-dir`    # to checkout if has fileinfo.so
    vim /path/to/php.ini
    	; Dynamic Extensions
    	extension = fileinfo.so
    
    # 6. Restart php-fpm
    service php5-fpm restart
```

### 源码安装第三方扩展

其实和安装官方扩展差不多，不同之处只是第三方扩展的源码包没有在 PHP 官方源码包中，需要手动到 `github` 或者 `pecl.php.net` 等下载，后续的配置、编译和安装完全一样。

```
    git clone https://github.com/mongodb/mongo-php-driver
    cd mongo-php-driver
    
    # Same with above
```

### 部分操作解释

- `phpize`准备扩展程序的文件夹以进行补充。它允许您通过创建`configure`文件来执行后续命令，并且基本上使扩展的文件夹“认为”它是PHP本身。`phpize`事实上，后面的过程与从源代码安装PHP时所做的相同 - 仅在这种情况下，只需编译PHP片段并准备好与已编译和安装的PHP一起使用。

- `./configure --enable-XXX`配置编译环境。它准备了编译器制作`XXX.so`我们将要使用的文件所需的一切。`enable-XXX`即使我们在`XXX`文件夹中，该标志也是必要的，因为该文件夹实际上认为它是 PHP，我们需要帮助它实现这种错觉。这个命令告诉它：“好的，你是PHP的源代码。现在使用intl扩展编译和安装。“，实际上，它是唯一可以从此文件夹安装的部分。

- `make`将源`XXX.so`文件编译成子文件夹，将文件放入您当前所在的`modules`文件夹中。

- `sudo make install` 将此文件移动到当前PHP安装的`extensions`文件夹中。

> - `phpize` prepares the extension’s folder for compliation. It allows you to do the subsequent commands by creating a configure file, and basically making the extension’s folder “think” it’s PHP itself. The procedure after phpize is, in fact, identical to what you would do when installing PHP from source – only in this case, just a fragment of PHP is compiled and prepared for use with the already compiled and installed PHP.
> - `./configure --enable-XXX` configures the environment for compilation. It prepares everything the compiler will need to craft the XXX.so file which we’ll be using. The enable-XXX flag is necessary even though we’re in the XXXfolder because the folder, effectively, thinks it is PHP, and we need to help it live that illusion. This command tells it: “Ok, you’re PHP’s source code. Now compile and install with the intl extension.”, when in fact, it’s the only part that can be installed from this folder.
> - `make` will compile the sources into XXX.so, placing the file into the very folder you’re currently in, under the modulessubfolder.
> - `sudo make install` will move this file into the current PHP installation’s extensions folder.

# PHP 版本切换

## phpbrew

Github: <u>[phpbrew](https://github.com/phpbrew/phpbrew)</u>

## phpenv

GitHub: <u>[phpenv](https://github.com/CHH/phpenv)</u>。

## php-version

GitHub：<u>[php-version](https://github.com/wilmoore/php-version)</u>。

## php-osx.liip.ch

官网：<u>[PHP 5.3 to 7.1 for OS X / macOS 10.6 to 10.12 as binary package](https://php-osx.liip.ch/)</u>。 

```
    curl -s https://php-osx.liip.ch/install.sh | bash -s 7.3
    curl -s https://php-osx.liip.ch/install.sh | bash -s 7.2
    curl -s https://php-osx.liip.ch/install.sh | bash -s 5.6
```

# 其他

## <u>[pickle](https://github.com/FriendsOfPhp/pickle)</u>

跨平台的 PHP 扩展安装工具，详见 [GitHub](https://github.com/FriendsOfPhp/pickle)。

```
    sudo pickle.phar install vld
```

## 懒人必备：Oneinstack

使用很简单，阿里云有些 ECS 就是用这个搭建的环境。

按照 <u>[官网](https://oneinstack.com/install/)</u>做就行了，问你安装什么你选择就是了，对于纯 PHP 项目来说一般只需要先安装基本的 LNMP 就行了。

```
    # CentOS
    yum -y install wget screen curl python
    # Debian
    apt-get -y install wget screen curl python
    # 阿里云用户下载
    wget http://aliyun-oss.linuxeye.com/oneinstack-full.tar.gz
    # 包含源码，国内外均可下载
    wget http://mirrors.linuxeye.com/oneinstack-full.tar.gz
    # 不包含源码，建议仅国外主机下载
    wget http://mirrors.linuxeye.com/oneinstack.tar.gz
    tar xzf oneinstack-full.tar.gz
    # 如果需要修改目录(安装、数据存储、Nginx日志)，请修改options.conf文件
    cd oneinstack
    # 如果网路出现中断，可以执行命令`screen -r oneinstack`重新连接安装窗口
    screen -S oneinstack
    ./install.sh     # 注：请勿sh install.sh或者bash install.sh这样执行
```

以下是常用的脚本和文件：

- `vhost.sh`：创建虚拟主机
- `addons.sh`：添加常用扩展。
- `version.txt`：这里面保存的是 `Oneinstack` 安装下载时会寻找的版本，如果安装前想要手动指定某个软件包的版本，可以直接修改这个文件。
- `upgrade.sh`：更新软件包，也可以用于切换 PHP 版本。

## All-in-one 集成环境

- Linux：<u>[Oneinstack](https://oneinstack.com/download/)</u>
- macOS：自带 && <u>[MAMPs](https://www.mamp.info/en/)</u>
- Windows：<u>[WAMP](https://sourceforge.net/projects/wampserver/)</u> or <u>[phpStudy](http://phpstudy.php.cn/)</u>

# FAQ

- 安装部分扩展时候编译成功安装到 `php-config --extension-dir` 时不能写入？

> 必须像Dan Willis建议的那样禁用系统完整性保护。
>
> 1. 重启你的电脑。
> 2. 当屏幕变黑时按住 `command+r` 直到您启动恢复。您将看到`OS X Utilities`菜单。
> 3. 下一个开放终端。然后输入以下命令：`csrutil disable` 
>    确保将其写下来以便记住它。
> 4. 重启你的mac，让它正常启动。
> 5. 再次打开终端并再次安装INTL：`sudo pecl install intl`
>    它现在将完成安装。
 

# 参考

- <u>[What are differences between PECL and PEAR?](https://stackoverflow.com/questions/1385346/what-are-differences-between-pecl-and-pear)</u>
- <u>[What is the difference between PEAR and Composer? [closed]](https://stackoverflow.com/questions/34199824/what-is-the-difference-between-pear-and-composer)</u>
- <u>[Failed to write error when installing intl extension on Os x El Capitan](https://stackoverflow.com/questions/33220553/failed-to-write-error-when-installing-intl-extension-on-os-x-el-capitan)</u>
- <u>[Checking if PEAR works](https://pear.php.net/manual/en/installation.checking.php)</u>
- <u>[和 PHP 相关的「安装」](http://blog.cjli.info/2017/01/12/All-About-Installations-Relate-To-PHP/)</u>