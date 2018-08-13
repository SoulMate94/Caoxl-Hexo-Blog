---
title: LNMP 配置教程
date: 2018-01-24 15:27:40
categories: www
tags: [LNMP, Linux, Nginx]
---

> 由于本人比较懒,使用LNMP搭建的个人网站,所以有必要写一下LNMP配置文档

<!--more-->

# 版本差异

一般情况下每个虚拟主机就是一个网站，网站一般通过域名进行访问。
本文为教程适合LNMP 1.2+，各个版本的添加过程基本类似，按提示操作即可。1.4版添加了SSL选项可以选择Letsencrypt和自备SSL证书，多PHP版本选择等功能。1.3版增加了FTP和数据库的创建等。LNMP 1.4的跨目录同时增加在fastcgi.conf中进行管控，LNMP1.2的防跨目录也由原来在php.ini中设置移至网站根目录下的.user.ini 进行控制。

LNMP 1.1及之前的版本采用/root/vhost.sh 进行添加虚拟主机。

LNMP 1.2开始使用lnmp命令进行管理，具体可以参看<u>[更新记录](https://lnmp.org/changelog.html)</u>

之前版本的LNMP都可以升级到新版的lnmp管理脚本，<u>[升级到1.4教程](https://lnmp.org/faq/upgrade1-4.html)</u>

# 网站搭建

## 添加网站(虚拟主机)

如果输入有错误需要删除时，可以按住`Ctrl`再按`Backspace` [←]键进行删除。

执行：**lnmp vhost add** 出现如下界面：

![lnmp vhost add](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-1.png)

这里要输入要添加网站的域名，我们已添加`www.vpser.net`域名为例，如上图提示后输入域名 `www.vpser.net` 回车后提示

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-2.png)

这里询问是否添加更多域名，直接再输入要绑定的域名，这里我们将 vpser.net 也绑上，多个域名空格隔开，如不需要绑其他域名就直接回车。

*(注：带www和不带www的是不同的域名，如需带www和不带的www的域名都访问同一个网站需要同时都绑定)。*

下面需要设置网站的目录

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-3.png)

网站目录不存在的话会创建目录。也可以输入已经存在的目录或要设置的目录（**注意如要输入必须是全路径即以/开头的完整路径！！！**）。不输入直接回车的话，采用默认目录：`/home/wwwroot/` 域名

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-4.png)

伪静态可以使URL更加简洁也利于SEO，如程序支持并且需要设置伪静态的话，如启用输入 `y` ，不启用输入 `n` 回车(注意LNMPA或LAMP模式没有该选择项！)。

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-5.png)

默认已经有了discuz、discuzx、discuzx2(Discuz X二级目录)、wordpress、wp2(WordPress二级目录)、typecho、typecho2(Typecho二级目录)、sablog、emlog、dabr、phpwind、、dedecms、drupal、ecshop、shopex等常用的Nginx伪静态配置文件，可以直接输入名称进行使用，如果是二级目录则需要对应配置文件里的二级目录的名称。

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-6.png)

这一步是设置日志，如启用日志输入 `y` ，不启用输入 `n` 回车。

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-7.png)

如果启用需要再输入要设置的日志的名称，默认日志目录为：`/home/wwwlogs/` 默认文件名为：**域名.log** 回车确认后，会询问是否添加数据库和数据库用户。

如果需要添加数据库输入 y ，不添加数据库输入 n 回车。

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-9.png)

如果要添加，需要先验证MySQL的 `root` 密码(注：输入密码将不显示)
提示 `Enter database name:` 后输入要创建的数据库名称，要创建的数据库用户名会和数据库同名，回车确认。

提示 `Please enter password for mysql user` 数据库名: 后输入要设置的密码，回车确认。

**如果安装了FTP服务器**会询问是否添加FTP账号

如果需要添加输入 y ，不添加输入 n 回车。

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-11.png)

提示`Enter ftp account name:` 后输入要创建的FTP账号名称，回车确认。
提示`Enter password for ftp account` FTP账号: 后输入要设置的密码，回车确认。

接下来是1.4新增的添加SSL功能

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-12.png)

如果需要添加输入 y ，不添加输入 n 回车。
选择了添加SSL会提示

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-13.png)

**有两个选项**

1 选项为使用自己准备好的SSL证书和key。

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-14.png)

> > 提示Please enter full path to SSL Certificate file 后输入要SSL证书的完整路径和文件名，回车确认。
> > 提示Please enter full path to SSL Certificate Key file: 后输入输入要key文件的完整路径和文件名，回车确认。

2 选项为使用免费SSL证书提供商`Letsencrypt`的证书，自动生成SSL证书等信息。

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-add-15.png)

需要输入一个邮箱回车确认。

提示 `Press any key to start create virtul host...` 后，回车确认便会开始创建虚拟主机。

添加成功会提示添加的域名、目录、伪静态、日志、数据库、FTP等相关信息，如下图：

![](https://lnmp.org/images/1.4/lnmp1.4-vhost-info.png)

## 列出网站(虚拟主机)

执行：**lnmp vhost list**

![](https://lnmp.org/images/1.2/lnmp-1.2-vhost-list.png)

## 删除网站(虚拟主机)

执行：**lnmp vhost del**

![](https://lnmp.org/images/1.2/lnmp-1.2-vhost-del.png)

删除网站会先列出当前已有虚拟主机，按提示输入要删除的虚拟主机域名 回车确认。

这里只是删除虚拟主机配置文件，网站文件并不会删除需要自己删除。
LNMP 1.2下需要执行：`chattr -i /网站目录/.user.ini` 后才能删除网站目录。

当执行chown或chmod对网站目录属主属组或权限进行操作时可能会提示`chown: changing ownership of `/home/wwwroot/default/.user.ini': Operation not permitted，`**不需要理会**，如果有强迫症可以参考前面先进行`chattr -i`的操作。

## 配置文件(虚拟主机)

- `LNMP`默认网站配置文件：/usr/local/nginx/conf/nginx.conf
- `LNMPA`默认网站配置文件：/usr/local/nginx/conf/nginx.conf 和 /usr/local/apache/conf/extra/httpd-vhosts.conf
- `LAMP`默认网站配置文件：/usr/local/apache/conf/extra/httpd-vhosts.conf

# 网站优化

## 伪静态管理

`LNMPA`或`LAMP`可以直接使用网站根目录下放`.htaccess` 来设置伪静态规则(具体规则可以去程序官网网站找google百度)，

但是在`LNMP`下，需要使用`Nginx`伪静态规则。

伪静态可以随时添加或删除，如果添加完虚拟主机后忘记或没有添加伪静态，可以通过修改配置文件来添加伪静态。
虚拟主机配置文件在：`/usr/local/nginx/conf/vhost/域名.conf`

> 如: /usr/local/nginx/conf/vhost/caoxl.com.conf

伪静态规则文件需要放在 `/usr/local/nginx/conf/` 下面。
编辑虚拟主机配置文件，可以使用 `vi`、`nano`或`winscp`，后2个工具对新手来说简单些。

例如前面我们添加的虚拟主机，打开后前半部分配置会显示如下：

![](https://lnmp.org/images/lnmp-rewrite-modify.png)

在root /home/wwwroot/www.vpser.net;这一行下面添加：

```
    include wordpress.conf;
```

上面的`wordpress.conf`为伪静态文件，如需要其他伪静态文件自己创建个并上传到`/usr/local/nginx/conf/` 下面并`include` **伪静态.conf**; 加完保存，执行：`/etc/init.d/nginx restart` 重启生效，如果报错可能是添加有误或伪静态规则有误。

## 上传网站程序

如果已经 <u>[安装FTP服务器](http://lnmp.org/faq/ftpserver.html)</u>可以直接使用ftp客户端通过你的`FTP`信息登录后上传网站或`sftp`等软件上传网站，设置好相关权限开始安装即可。

上传网站后建议执行：**chown www:www -R /path/to/dir** 对网站目录进行权限设置，`/path/to/dir`替换为你网站目录。
为了安全可以将一些不需要PHP运行的上传文件之类的目录去掉执行权限，参考：
<u>http://www.vpser.net/security/lnmp-remove-nginx-php-execute.html</u>

## 添加ssl证书开启https

对于已存在的虚拟主机添加https站点，可以执行：`lnmp ssl add` 命令添加ssl证书

目前有两种方式:一种是使用自备的ssl证书，二是采用**Let's Encrypt**的免费证书。

添加过程和前面的添加虚拟主机的过程是一样的，只是会多一项填写ssl证书和key的步骤或直接选择Let'sEncrypt自动生成证书。


## 防跨目录设置

LNMP 1.1及之前的版本使用php.ini里面，<u>[open_basedir](http://www.vpser.net/security/lnmp-cross-site-corss-dir-security.html)</u>设置

LNMP 1.2及更高版本防跨目录功能使用`.user.ini`，该文件在网站根目录下，可以修改`.user.ini` 里面的`open_basedir`的值来设置限制访问的目录或删除来移除防跨目录的设置。

`.user.ini`文件无法直接修改，如要修或删除需要先执行：`chattr -i /网站目录/.user.ini`

可以使用`winscp文件管理`、`vim编辑器`或`nano编辑器进`行修改。
 
删除的话 `rm -f /网站目录/.user.ini` 就可以。
修改完成后再执行：`chattr +i /网站目录/.user.ini`
`.user.ini`不需要重启一般5分钟左右生效，也可以重启一下`php-fpm`立即生效。
如果要更改网站目录必须要按上述方法修改防跨目录的设置，否则肯定报错！！

LNMP 1.4上如果不想用防跨目录或者修改`.user.ini`的防跨目录的目录还需要将 `/usr/local/nginx/conf/fastcgi.conf` 里面的 `fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/:/tmp/:/proc/"`; 在该行行前添加 `#` 或删除改行，**需要重启nginx**。

LNMP 1.4上也可以直接使用 `lnmp1.4/tools/` 目录下的 `./remove_open_basedir_restriction.sh` 进行移除。
**在Thinkphp、codeigniter、Laravel等框架下，网站目录一般是在public下，但是public下的程序要跨目录调用public上级目录下的文件，因为LNMP默认是不允许跨目录访问的，所以都是必须要将防跨目录访问的设置去掉，有时候这些框架类的程序提示500错误也可能是这个问题引起的。**

LNMPA或LAMP 模式1.2版本以上的防跨目录的设置使用的对应apache虚拟主机配置文件（lnmp管理工具添加的话文件是 `/usr/local/apache/conf/vhost/域名.conf` ）里的 `php_admin_value open_basedir` 参数进行设置。如果不需要设置可以在前面加 `#` 进行注释，或自行修改目录的限制。
重启apache生效。

## pathinfo设置

LNMP上各个版本pathinfo各个版本的设置基本一样：

lnmp v1.1上，修改对应虚拟主机的配置文件(`/usr/local/nginx/conf/vhost/域名.conf`)
**去掉 #include pathinfo.conf前面的#**，把`try_files $uri =404`; 前面加上 `#` 注释掉。

1.2,1.3上，修改对应虚拟主机的配置文件(`/usr/local/nginx/conf/vhost/域名.conf`)
将`include enable-php.conf;`替换为`include enable-php-pathinfo.conf;`

修改 `pathinfo` 需要**重启nginx**生效。


# 数据库管理

1.3以上版本，可以在添加虚拟主机时选择创建数据库，也可以单独使用 `lnmp database add` 按提示添加数据库，添加的用户名和数据库名是同名的。

**基础命令:**
 - **添加数据库命令**：`lnmp database add`
 - **编辑数据库用户密码命令**：`lnmp database edit`
 - **删除数据库命令**：`lnmp database del`
 - **列出所有数据库命令**：`lnmp database list`

# 其他

## 更新Let'sEncrypt SSL证书续期规则

因`Let's Encrypt`的`certbot`程序更新，参数发生些变化，可能导致**SSL证书续期失败**，建议8月23日前安装LNMP的用户更新一下`crontab`规则和`lnmp`管理脚本，自动更新命令：

```
    wget -O - http://soft.vpser.net/lnmp/ext/fix_renewssl.sh|bash
```

其他方法手动进行升级，执行：

```
    cd /tmp && wget http://soft.vpser.net/lnmp/lnmp1.4.tar.gz -O lnmp1.4.tar.gz && tar zxf lnmp1.4.tar.gz && cd lnmp1.4 && ./upgrade1.x-1.4.sh
```

升级lnmp管理脚本后再自行参考 <u>[crontab教程](https://www.vpser.net/manage/crontab.html)</u>，删除原certbot的规则

- **LNMP/LNMPA**模式添加上

```
    0 3 */7 * * /bin/certbot renew --disable-hook-validation --renew-hook "/etc/init.d/nginx reload"
```

- **LAMP**模式添加上

```
    0 3 */7 * * /bin/certbot renew --disable-hook-validation --renew-hook "/etc/init.d/httpd restart"
```

手动更新的话建议再执行：

```
    /bin/certbot renew --disable-hook-validation --renew-hook "/etc/init.d/nginx reload"
```

**LAMP模式执行：**

```
    /bin/certbot renew --disable-hook-validation --renew-hook "/etc/init.d/httpd restart"
```
 看一下能否正常更新。

# 参考

- [LNMP添加、删除虚拟主机及伪静态使用教程](https://lnmp.org/faq/lnmp-vhost-add-howto.html)

- [建议用户更新Let'sEncrypt SSL证书续期规则](https://lnmp.org/notice/fix-certbot-renew.html)