---
title: Linux 下安装开发环境(多版本PHP)
date: 2018-07-10 09:09:44
categories: Linux
tags: [Linux]
---


> `Linux`下配置一个 `PHP`+`Nginx`+`MySQL`/`MariaDB`环境

<!-- more -->

# 编译安装多版本`PHP`

```shell
    #!/bin/sh
    #编译安装多版本PHP, Nginx+PHP使用
    
    #定义函数，默认绿色输出 '#' 开头为红色
    function echocolor()
    {
        [[ $1 = '#' ]] && echo -e "\033[31m $* \033[0m" || echo -e "\033[32m $* \033[0m"
    }
    
    #检测网络
    ping baidu.com -c 2 &>/dev/null || ping qq.com -c 2 &>/dev/null || { echocolor '#' "网络异常！";exit; }
    
    #设置yum 安装包不删除 目录 /var/cache/yum/
    #sed  -i 's/keepcache=0/keepcache=1/g' /etc/yum.conf
    
    echocolor '#安装需要的库'
    yum install epel-release -y
    yum install gcc bison bison-devel zlib-devel libmcrypt-devel mcrypt mhash-devel openssl-devel libxml2-devel libcurl-devel bzip2-devel readline-devel libedit-devel sqlite-devel jemalloc jemalloc-devel libjpeg-devel libpng-devel libicu-devel openldap-devel ftp -y
    
    echocolor  "#设置相关目录"
    echocolor  "#下载目录"
    read Ddir
    
    echocolor "#安装版本"
    read Dver
    Sdir="/usr/local/php/php$Dver"
    
    echocolor "#安装目录:"
    echocolor "$Sdir"
    [ -d $Ddir ] || mkdir -p $Ddir
    [ -d $Sdir/etc ] || mkdir -p $Sdir/etc

    #[ -d $Sdir/apache/bin/apxs ] || mkdir -p $Sdir/apache/bin/apxs
    
    ##下载
    echocolor "#请输入下载链接"
    read Durl
    wget -c $Durl -O $Ddir/php.tar.gz
    
    echocolor "添加用户"
    useradd -s /sbin/nologin -M www
    id www
    
    echocolor  "解压"
    [ -e $Ddir/php.tar.gz ] && { tar xzvf $Ddir/php*.tar.gz -C $Ddir/;cd $Ddir/php*; } || { echocolor '#' "$Ddir目录无php源码包";exit ; }
    # [ `uname -m` = "x86_64" ] && LIB=/usr/lib64 || LIB=/usr/lib64
    # --with-apxs2=/usr/local/apache/bin/apxs #编译参数，apache+php使用
    make clean
    
    echocolor 编译
    ./configure \
    --prefix=$Sdir/ \
    --with-config-file-path=$Sdir/etc \
    --enable-inline-optimization \
    --disable-debug \
    --disable-rpath \
    --enable-shared \
    --enable-opcache \
    --enable-fpm \
    --with-fpm-user=www \
    --with-fpm-group=www \
    --with-mysql=mysqlnd \
    --with-mysqli=mysqlnd \
    --with-pdo-mysql=mysqlnd \
    --with-gettext \
    --enable-mbstring \
    --with-iconv \
    --with-mcrypt \
    --with-mhash \
    --with-openssl \
    --enable-bcmath \
    --enable-soap \
    --with-libxml-dir \
    --enable-pcntl \
    --enable-shmop \
    --enable-sysvmsg \
    --enable-sysvsem \
    --enable-sysvshm \
    --enable-sockets \
    --with-curl \
    --with-zlib \
    --enable-zip \
    --with-bz2 \
    --enable-ftp \
    --with-ldap-dir=/usr/lib \
    --with-png-dir=/usr/lib \
    --with-jpeg-dir=/usr/lib \
    --with-readline
    [ $? = 0 ] || { echocolor '#' "编译出现问题 ！";exit; }
    echo 编译安装
    make -j4 
    make install
    make clean
    make clean all
    echo #配置PHP
    cp php.ini-production $Sdir/etc/php.ini
    cp $Sdir/etc/php-fpm.conf.default $Sdir/etc/php-fpm.conf
    cp sapi/fpm/init.d.php-fpm $Sdir/bin/php-fpm-$Dver
    cp $Sdir/etc/php-fpm.d/www.conf.default $Sdir/etc/php-fpm.d/www.conf
    chmod +x $Sdir/bin/php-fpm-$Dver
    ln -s $Sdir/bin/php-fpm-$Dver /etc/init.d/php-fpm-$Dver
    ln -s $Sdir/etc/php-fpm.conf /etc/php-fpm/php-fpm-$Dver.conf
    ln -s $Sdir/etc/php.ini /etc/php/php-$Dver.ini
    chkconfig --add php-fpm-$Dver
    chkconfig php-fpm-$Dver on
    chkconfig --list php-fpm-$Dver
    echocolor #显示版本
    $Sdir/bin/php -v
    netstat -antp|grep php-fpm-$Dver
    echocolor "#php-$Dver已经安装完毕，开始安装mariadb"
    echocolor "#安装mariadb环境"
    yum -y install readline-devel zlib-devel openssl-devel libaio-devel
    echocolor "#安装cmake"
    yum -y install gcc-c++ ncurses-devel
    mkdir -p /var/tmp/cmake
    wget https://cmake.org/files/v3.11/cmake-3.11.1.tar.gz
    tar -zxv -f cmake-3.11.1.tar.gz
    cd cmake-3.11*
    ./configure
    make
    make install
```

## 说明

- [PHP安装包(tar.gz)](http://cn2.php.net/get/php-7.2.7.tar.gz/from/this/mirror)

> `7.2`: http://cn2.php.net/get/php-7.2.7.tar.gz/from/this/mirror
> `5.6`: http://cn2.php.net/get/php-5.6.36.tar.gz/from/this/mirror

**变量说明:**

- `Ddir`:  下载目录
- `Dver`:  安装版本
- `Sdir`:  安装目录
- `Durl`:  下载链接

**下载过程输入:**

- **设置目录** - 无需输入, 起提示作用
  - **下载目录** - 输入下载目录
  - **安装目录** - 输入安装目录 (默认前缀有`php`, 只需要输入类似`56`、`71`)
  - **请输入下载链接** - 输入下载链接(`tar.gz`包), 选择自己需要的`PHP`版本
  
```
     #设置相关目录 
     #下载目录 
     /root/linux
     
     #安装版本 
     72
     
     #安装目录: 
     /usr/local/php/php72
      
     #请输入下载链接 
     http://cn2.php.net/get/php-7.2.7.tar.gz/from/this/mirror
```

## 源码解释

```
    echocolor 编译
    ./configure \
    --prefix=$Sdir/ \                       # 指定 php 安装目录
    --with-config-file-path=$Sdir/etc \     # 指定 php.ini 位置
    --enable-inline-optimization \          # 优化线程
    --disable-debug \                       # 关闭调试模式
    --disable-rpath \                       # 关闭额外的运行库文件
    --enable-shared \
    --enable-opcache \
    --enable-fpm \                          # 打上PHP-fpm 补丁后才有这个参数，CGI方式安装的启动程序
    --with-fpm-user=www \
    --with-fpm-group=www \
    --with-mysql=mysqlnd \                  # mysql 安装目录，对`mysql`的支持
    --with-mysqli=mysqlnd \                 # 指定`mysqli`位置
    --with-pdo-mysql=mysqlnd \
    --with-gettext \                        # 打开gnu 的gettext 支持，编码库用到 
    --enable-mbstring \                     # 多字节，字符串的支持 
    --with-iconv \                          # 用于 PHP 编译时指定 iconv 在系统里的路径，否则会扫描默认路径
    --with-mcrypt \                         # mcrypt算法扩展
    --with-mhash \                          # mhash算法扩展
    --with-openssl \                        # openssl的支持，加密传输https时用到的
    --enable-bcmath \                       # 打开图片大小调整,用到zabbix监控的时候用到了这个模块
    --enable-soap \
    --with-libxml-dir \                     # 打开libxml2库的支持
    --enable-pcntl \                        # freeTDS需要用到的，可能是链接mssql 才用到
    --enable-shmop \                        # 这样就使得你的PHP系统可以处理相关的IPC函数了
    --enable-sysvmsg \                      # 这样就使得你的PHP系统可以处理相关的IPC函数了
    --enable-sysvsem \
    --enable-sysvshm \
    --enable-sockets \                      # 打开 sockets 支持
    --with-curl \                           # 打开curl浏览工具的支持
    --with-zlib \                           # 打开zlib库的支持，用于http压缩传输
    --enable-zip \                          # 打开对zip的支持
    --with-bz2 \                            # 打开对bz2文件的支持
    --enable-ftp \                          # 打开ftp的支持
    --with-ldap-dir=/usr/lib \
    --with-png-dir=/usr/lib \
    --with-jpeg-dir=/usr/lib \
    --with-readline
    [ $? = 0 ] || { echocolor '#' "编译出现问题 ！";exit; }
```

- 其他:

```
    --with-freetype-dir \                    # 打开对freetype字体库的支持
    --with-jpeg-dir \                        # 打开对jpeg图片的支持 
    --with-png-dir \                         # 打开对png图片的支持
    --with-curlwrappers \                    # 运用curl工具打开url流
    --with-gd \                              # 打开gd库的支持
    --enable-gd-native-ttf \                 # 支持TrueType字符串函数库
    --with-xmlrpc \                          # 打开xml-rpc的c语言
    --without-iconv \                        # 关闭iconv函数，字符集间的转换
    --with-ttf \                             # 打开freetype1.*的支持，可以不加了
    --with-xsl \                             # 打开XSLT 文件支持，扩展了libXML2库 ，需要libxslt软件 
    --with-pear \                            # 打开pear命令的支持，PHP扩展用的
    --enable-calendar \                      # 打开日历扩展功能 
    --enable-exif \                          # 图片的元数据支持
    --enable-magic-quotes \                  # 魔术引用的支持 
```

- `CGI`方式安装才用的参数

```
    --enable-fastCGI \                       # 支持fastcgi方式启动PHP
    --enable-force-CGI-redirect \            # 重定向方式启动PHP
    --with-ncurses \                         # 支持ncurses 屏幕绘制以及基于文本终端的图形互动功能的动态库
    --with-gmp \                             # 应该是支持一种规范
    --enable-dbase \                         # 建立DBA 作为共享模块
    --with-pcre-dir=/usr/local/bin/pcre-config      # perl的正则库案安装位置
    --disable-dmalloc
    --with-gdbm                              # dba的gdbm支持
    --enable-sigchild
    --enable-sysvshm
    --enable-zend-multibyte \                # 支持zend的多字节
    --enable-wddx
    --enable-soap
```

# 编译安装`Nginx`

## 环境准备

### 安装编译工具、依赖包

```
    yum -y install gcc gcc-c++ autoconf automake
    yum -y install zlib zlib-devel openssl openssl-devel pcre-devel
```

以上安装的是一些主要的依赖包，具体可根据自己情况或者报错信息提示安装或修改

### 新建匿名用户和用户组

```
    sudo groupadd -r nginx
    
    sudo useradd -s /sbin/nologin -g nginx -r nginx
```

## `Nginx`编译安装

### 下载源码包

> 建议从<u>[官网](https://nginx.org/download/)</u>下载

```
    wget https://nginx.org/download/nginx-1.15.1.tar.gz  # 版本自选
```

### 解压并编译

- 解压

```
    tar -zxvf nginx-1.15.1.tar.gz
```

- 配置

``` 
    ./configure 
    --prefix=/usr/local/nginx \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/lock/nginx.lock \
    --with-http_ssl_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_realip_module \
    --with-http_gzip_static_module \
    --with-http_stub_status_module \
    --with-mail \
    --with-mail_ssl_module \
    --with-debug \
    --http-client-body-temp-path=/var/tmp/nginx/client \
    --http-proxy-temp-path=/var/tmp/nginx/proxy \
    --http-fastcgi-temp-path=/var/tmp/nginx/fastcgi \
    --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi \
    --http-scgi-temp-path=/var/tmp/nginx/scgi \
```

配置正常的话可以看到这些信息

```
    Configuration summary
      + using system PCRE library
      + OpenSSL library is not used
      + md5: using system crypto library
      + sha1: using system crypto library
      + using system zlib library
    
      nginx path prefix: "/usr/local/nginx"
      nginx binary file: "/usr/local/nginx/sbin/nginx"
      nginx modules path: "/usr/local/nginx/modules"
      nginx configuration prefix: "/etc/nginx"
      nginx configuration file: "/etc/nginx/nginx.conf"
      nginx pid file: "/usr/local/nginx/logs/nginx.pid"
      nginx error log file: "/usr/local/nginx/logs/error.log"
      nginx http access log file: "/usr/local/nginx/logs/access.log"
      nginx http client request body temporary files: "client_body_temp"
      nginx http proxy temporary files: "proxy_temp"
      nginx http fastcgi temporary files: "fastcgi_temp"
      nginx http uwsgi temporary files: "uwsgi_temp"
      nginx http scgi temporary files: "scgi_temp"
```

以上编译参数只是配置了主要的东西，全部配置参数说明你可以通过这个命令查看: `./configure --help`

- 编译并安装

```
    make && make install
```

启动等命令,须进入到`/usr/local/nginx/sbin`目录下

- 启动:   `nginx`
- 停止:   `nginx -s stop`
- 重启:   `nginx -s reload`

# 编译安装`MySQL`

## 环境准备

### 编译环境

```
    yum install cmake
    yum install –y openssl openssl-devel ncurses ncurses-devel boost-devel
```

### 其他可选准备

```
    # 有旧版本的话,先卸载旧版本
    rpm -qa | grep mysql # 查看MySQL是否已经安装 (注意大小写，如果mysql 不行就换MySQL)
    
    [root@shuidianbang ~]# rpm -qa | grep mysql
    libmysqlclient16-5.1.69-1.w6.x86_64
    mysql55w-5.5.59-1.w6.x86_64
    mysql55w-libs-5.5.59-1.w6.x86_64
    mysql55w-server-5.5.59-1.w6.x86_64
    
    [root@shuidianbang ~]# rpm -qa | grep MySQL
    perl-DBD-MySQL-4.013-3.el6.x86_64
    
    // 屏幕上将显示已安装的mysql包名如: mysql55w-5.5.59-1.w6.x86_64 ; 
    rpm -e --nodeps mysql-server  (nodeps表示强制删除)
    rpm -e --nodeps mysql
    
    # 建立用户、组
    groupadd -r mysql
    useradd -r -g mysql -d /data/mysql -s /sbin/nologin mysql
    
    # 数据目录(个人习惯将MySQL数据目录改成/data/mysql,所以先准备好目录,在编译时指定该路径)
    mkdir /data/mysql
    chown -R mysql:mysql /data/mysql
```

### 相关编译选项

> <u>[完整的编译配置选项](https://dev.mysql.com/doc/refman/5.5/en/source-configuration-options.html)</u>

- 安装时路径相关的配置

```
    -DCMAKE_INSTALL_PREFIX=/usr/local/mysql     # 安装路径
    -DMYSQL_DATADIR=/data/mysql                 # MySQL的数据目录
    -DSYSCONFDIR=/etc                           # MySQL配置文件路径
```

- 存储引擎相关的配置

```
    # 若要明确指定编译某引擎，可使用类似如下的选项：
    -DWITH_INNOBASE_STORAGE_ENGINE=1
    -DWITH_ARCHIVE_STORAGE_ENGINE=1
    -DWITH_BLACKHOLE_STORAGE_ENGINE=1
    -DWITH_FEDEFATED_STORAGE_ENGINE=1
    
    # 若要明确指定不编译某引擎，可使用类似如下的选项：
    -DWITHOUT_<engine-name>_ENGINE=1
    
    # 如
    -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1
    -DWITHOUT_FEDERATED_STORAGE_ENGINE=1
    -DWITHOUT_PARTITION_STORAGE_ENGINE=1
```

- 其他配置项

```
    -DMYSQL_TCP_PORT=3306
    -DMYSQL_UNIX_ADDR=/tmp/mysql.sock
    -DENABLED_LOCAL_INFILE=1
    -DEXTRA_CHARSETS=all
    -DDEFAULT_CHARSET=utf8
    -DDEFAULT_COLLATION=utf8_general_ci
    -DWITH_DEBUG=0
    -DENABLED_PROFILING=1
    
    -DWITH_READLINE=1
    -DWITH_SSL=system
    -DWITH_ZLIB=system
    -DWITH_LIBWRAP=0
```

## 编译安装

> 注意：如果多次编译不成功的话,更改配置后记得`rm CMakeCache.txt`清理临时文件

- 下载源码

```
    wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.10.tar.gz
    tar -xvf mysql-5.7.10.tar.gz
    cd mysql-5.7.10
```

- 编译安装

```
    cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
            -DMYSQL_DATADIR=/data/mysql \
            -DSYSCONFDIR=/etc \
            -DWITH_INNOBASE_STORAGE_ENGINE=1 \
            -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
            -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
            -DWITH_READLINE=1 \
            -DWITH_SSL=system \
            -DWITH_ZLIB=system \
            -DWITH_LIBWRAP=0 \
            -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
            -DDEFAULT_CHARSET=utf8 \
            -DENABLE_DOWNLOADS=1 \
            -DDEFAULT_COLLATION=utf8_general_ci \
            -DWITH_BOOST=/usr/local/boost
            # -DDOWNLOAD_BOOST=1 \ 如果没有boost-devel时可以加上该选项
            
    make && make install
```

- 修改属组和属主

```
    chown -R mysql:mysql /usr/local/mysql/
```

## 配置和初始化

### 初始化

### 配置文件

**关于配置文件的一点说明:**

众所周知,`MySQL`至关重要的配置文件就是`my.cnf`。

该文件可以存在多个,`MySQL`启动时候的搜寻路径如下:

- `/etc/my.cnf`
- `/etc/mysql/my.cnf`
- `/${MYSQL_HOME}/my.cnf`
- `default-extra-file`
- `~/my.cnf`

**对于配置文件搜寻顺序的说明:**

- 1. 即使是在第一个位置找到了配置文件，后续路径中的搜寻仍然会继续
- 2. 如果找到多个配置文件，所有配置文件中的配置都将生效
- 3. 如果在多个文件中找到了同一个配置，将使用最后一个配置


**复制配置文件:**

```
    cp support-files/my-default.cnf /etc/my.cnf
    
    # 注意--如果复制的配置文件中没有配置以下几项 请手动指定
    # 否则mysql服务启动会找不到目录
    # /etc/my.cnf
    [mysqld]
    basedir =/usr/local/mysql
    datadir =/data/mysql
    port =3306
```

### 将`MySQL`做成系统服务

```
    # support-files在安装目录和解压后的MySQL目录都有
    
    cp support-files/mysql.server /etc/init.d/mysqld
    chkconfig --add mysqld
    chkconfig --list mysqld
```

### 将`MySQL`加入环境变量

```
    # 编辑/etc/profile.d/mysql.sh
    vim /etc/profile.d/mysql.sh
    
    # 加入如下文件
    export MYSQL_HOME=/usr/local/mysql
    export PATH=$PATH:$MYSQL_HOME/bin
    
    # 使配置生效
    source /etc/profile.d/mysql.sh
```

## 启动`MySQL`

- 注意,在这一步可能会遇到很多问题,请确保权限正常
  - `chown -R mysql:mysql /data/mysql/`
  - `chown -R mysql:mysql /usr/local/mysql` 
  - `chown -R mysql:mysql /usr/share/mysql/`

- 做成系统服务后直接启动即可

```
    service mysqld start
```

- `root`登录

```
    # 注意此处的密码是在初始化时生成的临时密码
    mysql -u root -p
    
    # 修改密码
    set password=password('新密码');
    
    # 刷新权限
    flush privileges;
```

# 编译安装`MariaDB`

> `MariaDB`数据库管理系统是`MySQL`的一个分支，主要由开源社区在维护，采用`GPL`授权许可。
开发这个分支的原因之一是：甲骨文公司收购了`MySQL`后，有将`MySQL`闭源的潜在风险，因此社区采用分支的方式来避开这个风险

## 环境准备

### 源码包下载

> 选择自己需要的版本: https://downloads.mariadb.org/

这里使用:[mariadb-10.2.16.tar.gz	](https://downloads.mariadb.org/interstitial/mariadb-10.2.16/source/mariadb-10.2.16.tar.gz/from/http%3A//mirrors.neusoft.edu.cn/mariadb/)

### 编译环境及依赖关系

```
    yum groupinstall -y Development Tools
    yum -y install ncurses-devel zlib-devel
```

- 获取`cmake`:

```
    wget https://cmake.org/files/v3.12/cmake-3.12.0-rc3.tar.gz
    tar -zxvf cmake-3.12.0-rc3.tar.gz -C /usr/local/
```

- 编译安装:

```
    cd /usr/local/cmake-3.12.0/
    ./configure 
```

## 安装`MariaDB`

```
    wget https://downloads.mariadb.org/interstitial/mariadb-10.2.16/source/mariadb-10.2.16.tar.gz/from/http%3A//mirrors.neusoft.edu.cn/mariadb/
    tar -zxvf mariadb-10.2.16.tar.gz -C /usr/local/mysql
    
    cd /usr/local/mysql
    
    cmake  -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
    -DSYSCONFDIR=/etc \
    -DDEFAULT_CHARSET=utf8 \
    -DDEFAULT_COLLATION=utf8_general_ci \
    -DWITH_EXTRA_CHARSETS=all \
    
    # 注:
    # 第一行是mysql主程序安装目录
    # 第二行是配置文件目录
    # 第三行默认字符集为utf8
    # 第四行默认的字符集校对规则
    # 第五行安装所有字符集
    
    # 最后
    make && make install
```

### 创建`mysql`用户

```
    useradd -M -s /sbin/nologin mysql
    chown mysql:root /usr/local/mysql/
```
### `MariaDB`配置文件创建及更改

```
    # 复制配置文件
    cp support-files/my-medium.cnf /etc/my.cnf
    
    # 复制启动脚本
    cp support-files/mysql.server  /etc/init.d/mysqld
    
    # 编辑配置文件,做以下修改
    vim /etc/my.conf 
    
    # 指定数据库路径，不然无法启动mysql
    datadir = /usr/local/msyql/data/
    
    # 设置后当创建数据库的表的时候表文件都会分离开，方便复制表，不开启创建的表都在一个文件
    innodb_file_per_table = on
    
    # 跳过名称反解，Mysql每次使用客户端链接时都会把ip地址反解成主机名
    skip_name_resolve = on  
```

### 设置`Mysql`环境变量

```
    echo "export PATH=$PATH:/usr/local/mysql/bin">>/etc/profile
```

### 执行脚本初始化数据库

```
    /usr/local/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/mydata/ 
```

### 安全初始化mysql及设定(前提先启动Mysql服务: `server mysqld start`)

```
    /usr/local/mysql/bin/mysql_secure_installation
```

# 常见问题

## `virtual memory exhausted: Cannot allocate memory`

发生该问题的原因是服务器的内存不够，从而导致编译失败。

而购买的`Linux`服务器，未给你分配虚拟内存，所以可以通过自行增加虚拟内存的方法予以解决

- 第一步:

```
    [root@izj6c6djex81rijczh0t8yz ~]# free -m
                  total        used        free      shared  buff/cache   available
    Mem:            991         142         664           2         184         696
    Swap:             0           0           0
```

- 第二步:

```
    [root@izj6c6djex81rijczh0t8yz ~]# mkdir /usr/img/
    [root@izj6c6djex81rijczh0t8yz ~]# rm -rf /usr/img/swap
    [root@izj6c6djex81rijczh0t8yz ~]# dd if=/dev/zero of=/usr/img/swap bs=1024 count=2048000
    
    2048000+0 records in
    2048000+0 records out
    2097152000 bytes (2.1 GB) copied, 36.0165 s, 58.2 MB/s
```

- 第三步:

```
    [root@izj6c6djex81rijczh0t8yz ~]# mkswap /usr/img/swap    
    Setting up swapspace version 1, size = 2047996 KiB
    no label, UUID=b2d350d1-b5e4-427b-ae10-6a523121b6b9
    
    [root@izj6c6djex81rijczh0t8yz ~]# swapon /usr/img/swap
    swapon: /usr/img/swap: insecure permissions 0644, 0600 suggested.

    [root@izj6c6djex81rijczh0t8yz ~]# free -m
                  total        used        free      shared  buff/cache   available
    Mem:            991         144          67           2         779         688
    Swap:          1999           0        1999
```

使用完毕后可以关掉`swap`

```
    [root@izj6c6djex81rijczh0t8yz ~]# swapoff swap
    [root@izj6c6djex81rijczh0t8yz ~]# rm -f /usr/img/swap
```