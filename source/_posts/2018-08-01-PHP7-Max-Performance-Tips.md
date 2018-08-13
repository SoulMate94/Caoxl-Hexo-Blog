---
title: 让PHP7达到最高性能的几个Tips
date: 2018-08-01 10:46:59
categories: PHP7
tags: [PHP, PHP7]
---

> 原文地址: `http://www.laruence.com/2015/12/04/3086.html`

<!-- more -->

这里只做博客的搬运工, 方便自己查找查看.

# 1. `Opcache`

记得启用`Zend Opcache`, 因为`PHP7`即使不启用`Opcache`速度也比`PHP-5.6`启用了`Opcache`快, 所以之前测试时期就发生了有人一直没有启用`Opcache`的事情. 启用`Opcache`非常简单, 在`php.ini`配置文件中加入:

```
    zend_extension=opcache.so
    opcache.enable=1
    opcache.enable_cli=1"
```

# 2. 使用新的编译器

使用新一点的编译器, 推荐`GCC 4.8`以上, 因为只有`GCC 4.8`以上PHP才会开启`Global Register for opline and execute_data`支持, 这个会带来`5%`左右的性能提升(`Wordpres`的`QPS`角度衡量)

其实`GCC 4.8`以前的版本也支持, 但是我们发现它支持的有`Bug`, 所以必须是`4.8`以上的版本才会开启这个特性.

```
    [root@caoxianliang ~]# gcc -v
    Using built-in specs.
    COLLECT_GCC=gcc
    COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-redhat-linux/4.8.5/lto-wrapper
    Target: x86_64-redhat-linux
    Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-bootstrap --enable-shared --enable-threads=posix --enable-checking=release --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-linker-hash-style=gnu --enable-languages=c,c++,objc,obj-c++,java,fortran,ada,go,lto --enable-plugin --enable-initfini-array --disable-libgcj --with-isl=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/isl-install --with-cloog=/builddir/build/BUILD/gcc-4.8.5-20150702/obj-x86_64-redhat-linux/cloog-install --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
    Thread model: posix
    gcc version 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) 
```

# 3. `HugePage`

我之前的文章也介绍过: [让你的PHP7更快之Hugepage](http://www.laruence.com/2015/10/02/3069.html) , 首先在系统中开启`HugePages`, 然后开启`Opcache`的`huge_code_pages`.

以我的`CentOS 6.5`为例, 通过:

```
    [root@caoxianliang ~]# sudo sysctl vm.nr_hugepages=512
    vm.nr_hugepages = 512
```

分配`512`个预留的大页内存:

```
    [root@caoxianliang ~]# cat /proc/meminfo | grep Huge
    AnonHugePages:     59392 kB
    HugePages_Total:      512
    HugePages_Free:       37
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
```

然后再`php.ini`中加入:

```
    opcache.huge_code_pages=1
```

> 这样一来, `PHP`会把自身的`text`段, 以及内存分配中的`huge`都采用大内存页来保存, 减少`TLB miss`, 从而提高性能.


# 4. `Opcache file cache`

开启`Opcache File Cache`(实验性), 通过开启这个, 我们可以让`Opcache`把`opcode`缓存缓存到外部文件中, 对于一些脚本, 会有很明显的性能提升.

- 在`php.ini`中加入:

```
    opcache.file_cache=/tmp
```

这样`PHP`就会在`/tmp`目录下`Cache`一些`Opcode`的二进制导出文件, 可以跨`PHP`生命周期存在.


# `PGO`

我之前的文章: [让你的PHP7更快(GCC PGO)](http://www.laruence.com/2015/06/19/3063.html) 也介绍过, 如果你的`PHP`是专门为一个项目服务, 比如只是为你的`Wordpress`, 或者`drupal`, 或者其他什么, 那么你就可以尝试通过`PGO`, 来提升`PHP`, 专门为你的这个项目提高性能.

具体的, 以`wordpress 4.1`为优化场景.. 首先在编译`PHP`的时候首先:

```
    make prof-gen
```

然后用你的项目训练`PHP`, 比如对于`Wordpress`:

```
    sapi/cgi/php-cgi -T 100 /home/huixinchen/local/www/htdocs/wordpress/index.php >/dev/null
```

也就是让`php-cgi`跑`100`遍`wordpress`的首页, 从而生成一些在这个过程中的`profile`信息.

最后:

```
    make prof-clean
    make prof-use && make install
```

这个时候你编译得到的`PHP7`就是为你的项目量身打造的最高性能的编译版本.
