---
title: Linux 命令 「curl」
date: 2018-06-11 10:10:58
categories: Linux cmd
tags: [Linux]
---

> **curl命令**是一个利用URL规则在命令行下工作的文件传输工具。
它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称curl为下载工具。

<!-- more -->

# 常见参数

- `-A`/`--user-agent <string>`           设置用户代理发送给服务器
- `-b`/`--cookie <name=string/file>`     cookie字符串或文件读取位置
- `-c`/`--cookie-jar <file>`             操作结束后吧cookie写入到这个文件中
- `-C`/`--continue-at <offset>`          断点续转
- `-D`/`--dump-header <file>`            把header信息写入该文件中
- `-e`/`--referer`                       来源地址
- `-f`/`--fail`                          连接失败时不显示http错误
- `-o`/`--output`                        把输出写到该文件中
- `-O`/`--remote-name`                   把输出写到该文件中,保留远程文件的文件名
- `-r`/`--range <range>`                 检索来自HTTP/1.1或FTP服务器字节范围
- `-s`/`--silent`                        静音模式,不输出任何东西
- `-T`/`--upload-file <file>`            上传文件
- `-u`/`--user <user[:password]>`        设置服务器的用户和密码
- `-w`/`--write-out [format]`            什么输出完成后
- `-x`/`--proxy <host [:port]>`          在给定的端口上使用HTTP代理
- `-#`/`--progress-bar`                  进度条显示当前的传送状态

# 常见用例

## 基本用法

```
    curl http://www.caoxl.com
```

## 自定义`header`

```
    curl -H "Referer:www.caoxl.com -H "User-agent: Custom-User-Agent" http://caxl.com
```

## 发送`POST`/`GET` 请求

- `POST`

```
    curl -d "username=caoxl&passwd=123456" -X POST http://caoxl.com/login
```

在使用 `-d` 的情况下，如果省略 `-X`，则默认为 `POST` 方式：

```
    curl -d "username=caoxl&passwd=123456" http://caoxl.com/login
```

- `GET`

```
    curl -d "somedata" -X GET http://caoxl.com/login
    
    // 或 -G
    curl -d "somedata" -G http://caoxl.com/api
```


## 显示`HTTP`头

```
    curl -I http://www.caoxl.com
    
    // 同时显示HTTP头和文件内容,使用 `-i`选项
    curl -i http://www.caoxl.com
```

## 保存访问的网页

1. 使用`linux`的重定向功能保存

```
    curl http://caoxl.com >> caoxl.html
```

2. 可以使用`curl`的内置`option:-o`(小写)保存网页

```
    curl -o caoxl2.html http://caoxl.com
```

3. 可以使用`curl`的内置`option:-O`(大写)保存网页中的文件

**要注意这里后面的url要具体到某个文件，不然抓不下来**

```
    curl -O http://caoxl.com/index.html
```

## 测试网页返回值

```
    curl -o /dev/null -s -w %{http_code} www.caoxl.com
    
    // 返回HTTP状态码
```
> 在脚本中，这是很常见的测试网站是否正常的用法

## 指定proxy服务器以及其端口

很多时候上网需要用到代理服务器(比如是使用代理服务器上网或者因为使用`curl`别人网站而被别人屏蔽IP地址的时候)，
幸运的是`curl`通过使用内置`option：-x`来支持设置代理

```
    curl -x 192.169.100.100:1080 http://caoxl.com
```

## 跟随链接重定向(`HTTPS`)

```
    curl -L http://caoxl.com
```

## `cookie`

有些网站是使用`cookie`来记录`session`信息。对于`chrome`这样的浏览器，可以轻易处理`cookie`信息，
但在`curl`中只要增加相关参数也是可以很容易的处理`cookie`

1. 保存`http`的`response`里面的`cookie`信息。内置`option:-c`（小写）

```
    curl -c cookiec.txt http://caoxl.com
    // 执行后cookie信息就被存到了cookiec.txt里面了, 无cookie则不会生成文件
```

2. 保存`http`的`response`里面的`header`信息。内置`option: -D`

```
    curl -D cookieD.txt http://caoxl.com
```

3. 使用`cookie`

很多网站都是通过监视你的`cookie`信息来判断你是否按规矩访问他们的网站的，
因此我们需要使用保存的`cookie`信息。内置`option: -b`

```
    curl -b cookieb.txt http://caoxl.com    
```

## 模仿浏览器

有些网站需要使用特定的浏览器去访问他们，有些还需要使用某些特定的版本。
`curl`内置`option:-A`可以让我们指定浏览器去访问网站

```
    curl -A "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.0)" http://caoxl.com
```

这样服务器端就会认为是使用`IE8.0`去访问的

## 伪造`referer`（盗链）

很多服务器会检查`http`访问的`referer`从而来控制访问。
比如：你是先访问首页，然后再访问首页中的邮箱页面，这里访问邮箱的`referer`地址就是访问首页成功后的页面地址，
如果服务器发现对邮箱页面访问的`referer`地址不是首页的地址，就断定那是个盗连了
`curl`中内置`option：-e`可以让我们设定`referer`

```
    curl -e "www.caoxl.com" http://blog.caoxl.com
```

> 这样就会让服务器以为你是从www.caoxl.com点击某个链接过来的

## 下载文件

1. 利用`curl`下载文件。

**使用内置`option：-o`(小写)**

```
    curl -o MasterSlave.jpg http://www.caoxl.com
```

**使用内置`option：-O`(大写)**

```
    curl -O http://www.caoxl.com/MasterSlave.jpg
    // 这样就会以服务器上的名称保存文件到本地
```

2. 循环下载

有时候下载图片可以能是前面的部分名称是一样的，就最后的尾椎名不一样

```
    curl -O http://caoxl.com/MasterSlave[1-5].jpg
```
这样就会把MasterSlave1,MasterSlave2,MasterSlave3,MasterSlave4,MasterSlave5全部保存

3. 下载重命名

```
    curl -O http://www.caoxl.com/{newname,newname2}/MasterSlave[1-5].jpg
```

这样在`newname/MasterSlave.jpg`的文件下载下来就会变成`newname_MasterSlave1.jpg`,
其他文件依此类推，从而有效的避免了文件被覆盖

```
    curl -o #1_#2.jpg http://caoxl.com/{newname,newname2}/MasterSlave[1-5].jpg
```

4. 分块下载

有时候下载的东西会比较大，这个时候我们可以分段下载。使用内置`option：-r`

```
    curl -r 0-100 -o part1.jpg http://www.caoxl.com/MasterSlave.jpg
    curl -r 1-200 -o part2.jpg http://www.caoxl.com/MasterSlave.jpg
    curl -r 200- -o part3.jpg http://www.caoxl.com/MasterSlave.jpg
    cat part* > NasterSlave.jpg
```

5. 通过ftp下载文件

`curl`可以通过`ftp`下载文件, `curl`提供两种从`ftp`中下载的语法

```
    curl -O -u 用户名:密码 ftp://www.caoxl.com/test.jpg
    
    // 或
    curl -O ftp://用户名:密码@www.caoxl.com/test.jpg
```

6. 显示下载进度条

```
    curl -# -O http://www.caoxl.com/test.jpg
```

7. 不会显示下载的进度信息

```
    curl -s -O http://www.caoxl.com/test.jpg
```

8. 同时下载多个文件

```
    curl -O http://www.caoxl/test1.jpg -O http://www.caoxl.com/test2.jpg
    
    // 或
    curl -o test.html http://caoxl.com/page/1/ -o http://caoxl.com/page/2/
```

## 断点续传

在windows中，我们可以使用迅雷这样的软件进行断点续传。
`curl`可以通过内置`option:-C`同样可以达到相同的效果
如果在下载`test.jpg`的过程中突然掉线了，可以使用以下的方式续传

```
    curl -C -O http://www.caoxl.com/test.jpg
```

## 上传文件

`curl`不仅仅可以下载文件，还可以上传文件。通过内置`option:-T`来实现

```
    curl -T test.jpg -u 用户名:密码 ftp://www.caoxl.com/
```

这样就向ftp服务器上传了文件`test.jpg`

## 显示抓取错误

```
    curl -f http://caoxl.com/error
```

# 其他参数

```
    -a/--append                     上传文件时，附加到目标文件
    --anyauth                       可以使用“任何”身份验证方法
    --basic                         使用HTTP基本验证
    -B/--use-ascii                  使用ASCII文本传输
    -d/--data <data>                HTTP POST方式传送数据
    --data-ascii <data>             以ascii的方式post数据
    --data-binary <data>            以二进制的方式post数据
    --negotiate                     使用HTTP身份验证
    --digest                        使用数字身份验证
    --disable-eprt                  禁止使用EPRT或LPRT
    --disable-epsv                  禁止使用EPSV
    --egd-file <file>               为随机数据(SSL)设置EGD socket路径
    --tcp-nodelay                   使用TCP_NODELAY选项
    -E/--cert <cert[:passwd]>       客户端证书文件和密码 (SSL)
    --cert-type <type>              证书文件类型 (DER/PEM/ENG) (SSL)
    --key <key>                     私钥文件名 (SSL)
    --key-type <type>               私钥文件类型 (DER/PEM/ENG) (SSL)
    --pass  <pass>                  私钥密码 (SSL)
    --engine <eng>                  加密引擎使用 (SSL). "--engine list" for list
    --cacert <file>                 CA证书 (SSL)
    --capath <directory>            CA目   (made using c_rehash) to verify peer against (SSL)
    --ciphers <list>                SSL密码
    --compressed                    要求返回是压缩的形势 (using deflate or gzip)
    --connect-timeout <seconds>     设置最大请求时间
    --create-dirs                   建立本地目录的目录层次结构
    --crlf                          上传是把LF转变成CRLF
    --ftp-create-dirs               如果远程目录不存在，创建远程目录
    --ftp-method [multicwd/nocwd/singlecwd]    控制CWD的使用
    --ftp-pasv                      使用 PASV/EPSV 代替端口
    --ftp-skip-pasv-ip              使用PASV的时候,忽略该IP地址
    --ftp-ssl                       尝试用 SSL/TLS 来进行ftp数据传输
    --ftp-ssl-reqd                  要求用 SSL/TLS 来进行ftp数据传输
    -F/--form <name=content>        模拟http表单提交数据
    -form-string <name=string>      模拟http表单提交数据
    -g/--globoff                    禁用网址序列和范围使用{}和[]
    -G/--get                        以get的方式来发送数据
    -h/--help                       帮助
    -H/--header <line>              自定义头信息传递给服务器
    --ignore-content-length         忽略的HTTP头信息的长度
    -i/--include                    输出时包括protocol头信息
    -I/--head                       只显示文档信息
    -j/--junk-session-cookies       读取文件时忽略session cookie
    --interface <interface>         使用指定网络接口/地址
    --krb4 <level>                  使用指定安全级别的krb4
    -k/--insecure                   允许不使用证书到SSL站点
    -K/--config                     指定的配置文件读取
    -l/--list-only                  列出ftp目录下的文件名称
    --limit-rate <rate>             设置传输速度
    --local-port<NUM>               强制使用本地端口号
    -m/--max-time <seconds>         设置最大传输时间
    --max-redirs <num>              设置最大读取的目录数
    --max-filesize <bytes>          设置最大下载的文件总量
    -M/--manual                     显示全手动
    -n/--netrc                      从netrc文件中读取用户名和密码
    --netrc-optional                使用 .netrc 或者 URL来覆盖-n
    --ntlm                          使用 HTTP NTLM 身份验证
    -N/--no-buffer                  禁用缓冲输出
    -p/--proxytunnel                使用HTTP代理
    --proxy-anyauth                 选择任一代理身份验证方法
    --proxy-basic                   在代理上使用基本身份验证
    --proxy-digest                  在代理上使用数字身份验证
    --proxy-ntlm                    在代理上使用ntlm身份验证
    -P/--ftp-port <address>         使用端口地址，而不是使用PASV
    -Q/--quote <cmd>                文件传输前，发送命令到服务器
    --range-file                    读取（SSL）的随机文件
    -R/--remote-time                在本地生成文件时，保留远程文件时间
    --retry <num>                   传输出现问题时，重试的次数
    --retry-delay <seconds>         传输出现问题时，设置重试间隔时间
    --retry-max-time <seconds>      传输出现问题时，设置最大重试时间
    -S/--show-error                 显示错误
    --socks4 <host[:port]>          用socks4代理给定主机和端口
    --socks5 <host[:port]>          用socks5代理给定主机和端口
    -t/--telnet-option <OPT=val>    Telnet选项设置
    --trace <file>                  对指定文件进行debug
    --trace-ascii <file>            Like --跟踪但没有hex输出
    --trace-time                    跟踪/详细输出时，添加时间戳
    --url <URL>                     Spet URL to work with
    -U/--proxy-user <user[:password]>  设置代理用户名和密码
    -V/--version                    显示版本信息
    -X/--request <command>          指定什么命令
    -y/--speed-time                 放弃限速所要的时间。默认为30
    -Y/--speed-limit                停止传输速度的限制，速度时间'秒
    -z/--time-cond                  传送时间设置
    -0/--http1.0                    使用HTTP 1.0
    -1/--tlsv1                      使用TLSv1（SSL）
    -2/--sslv2                      使用SSLv2的（SSL）
    -3/--sslv3                      使用的SSLv3（SSL）
    --3p-quote                      like -Q for the source URL for 3rd party transfer
    --3p-url                        使用url，进行第三方传送
    --3p-user                       使用用户名和密码，进行第三方传送
    -4/--ipv4                       使用IP4
    -6/--ipv6                       使用IP6
```

# 参考

- [Linux curl命令详解](http://www.cnblogs.com/duhuo/p/5695256.html)