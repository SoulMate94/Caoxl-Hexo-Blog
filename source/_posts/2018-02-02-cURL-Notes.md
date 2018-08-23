---
title: cURL 「使用笔记」
date: 2018-02-02 17:23:57
categories: Internet
tags: [cURL, Linux, Internet]
---

> curl是利用URL语法在命令行方式下工作的开源文件传输工具。它被广泛应用在Unix、多种Linux发行版中，并且有DOS和Win32、Win64下的移植版本。

<!--more-->


# cURL 常用命令

## Download a Single File

**下载单个文件**，默认将输出打印到标准输出中(STDOUT)中

```
    curl http://www.centos.org
    
    # 同样可以使用转向字符">"对输出进行转向输出
    curl http://www.centos.org > centos-org.html
```

## Save the cURL Output to a file

通过`-o`/`-O`选项**保存下载的文件到指定的文件中**：

- `-o`：将文件保存为命令行中指定的文件名的文件中
- `-O`：使用URL中默认的文件名保存文件到本地

```
    # 将文件下载到本地并命名为mygettext.html
    curl -o mygettext.html http://www.gnu.org/software/gettext/manual/gettext.html
    
    # 将文件保存到本地并命名为gettext.html
    curl -O http://www.gnu.org/software/gettext/manual/gettext.html
```

## Fetch Multiple Files at a time

**同时获取多个文件**

```
    curl -O URL1 -O URL2
    
    curl -O http://www.gnu.org/software/gettext/manual/html_node/index.html -O http://www.gnu.org/software/gettext/manual/gettext.html
```

## Follow HTTP Location Headers with -L option
    
 默认情况下CURL不会发送 `HTTP Location headers`(重定向).当一个被请求页面移动到另一个站点时，会发送一个 `HTTP Loaction header` 作为请求，然后将请求重定向到新的地址上。
 
```
    curl http://www.google.com
    
    <TITLE>302 Moved</TITLE>
    <H1>302 Moved</H1>
    The document has moved
    <A HREF="http://www.google.co.in/">here</A>
    
    # 通过-L选项进行重定向
    curl -L http://www.google.com
```

## Continue/Resume a Previous Download

**断点续传**

通过使用-C选项可对大文件使用断点续传功能，如：

```
    # 当文件在下载完成之前结束该进程
    curl -O http://www.gnu.org/software/gettext/manual/gettext.html
    ##############             20.1%
    
    # 通过添加-C选项继续对该文件进行下载，已经下载过的文件不会被重新下载
    curl -C - -O http://www.gnu.org/software/gettext/manual/gettext.html
    ###############            21.1%
```

## Limit the Rate of Data Transfer

**对CURL使用网络限速**

通过--limit-rate选项对CURL的最大网络使用进行限制

```
    # 下载速度最大不会超过1000B/second
    
    curl --limit-rate 1000B -O http://www.gnu.org/software/gettext/manual/gettext.html
```

## Download a file only if it is modified before/after the given time

**下载指定时间内修改过的文件**

```
    curl -z 31-Dec-18 http://www.example.com/yy.html
    
    curl -z -31-Dec-18 http://www.example.com/yy.html
```

## Pass HTTP Authentication in cURL

**CURL授权**

在访问需要授权的页面时，可通过-u选项提供用户名和密码进行授权

```
    curl -u username:password URL
    
    #  通常的做法是在命令行只输入用户名，之后会提示输入密码，这样可以保证在查看历史记录时不会将密码泄露
    curl -u username URL
```

## Download Files from FTP server

**从FTP服务器下载文件**

CURL同样支持FTP下载，若在url中指定的是某个文件路径而非具体的某个要下载的文件名，CURL则会列出该目录下的所有文件名而并非下载该目录下的所有文件

```
    # 列出public_html下的所有文件夹和文件
    curl -u ftpuser:ftppass -O ftp://ftp_server/public_html/
   
    
    # 下载xss.php文件
     curl -u ftpuser:ftppass -O ftp://ftp_server/public_html/xss.php
```

## List/Download using Ranges

```
    curl   ftp://ftp.uk.debian.org/debian/pool/main/[a-z]/
```

## Upload Files to FTP Server

**上传文件到FTP服务器**

通过 -T 选项可将指定的本地文件上传到FTP服务器上

```
    # 将myfile.txt文件上传到服务器
    curl -u ftpuser:ftppass -T myfile.txt ftp://ftp.testserver.com
    
    # 同时上传多个文件
    curl -u ftpuser:ftppass -T "{file1,file2}" ftp://ftp.testserver.com
    
    # 从标准输入获取内容保存到服务器指定的文件中
    curl -u ftpuser:ftppass -T - ftp://ftp.testserver.com/myfile_1.txt
```

## More Information using Verbose and Trace Option

**获取更多信息**

通过使用 -v 和 -trace获取更多的链接信息

```
    curl -v http://google.co.in
```

## Get Definition of a Word using DICT Protocol

**通过字典查询单词**

```
    # 查询bash单词的含义
    curl dict://dict.org/d:bash
    
    # 列出所有可用词典
    curl dict://dict.org/show:db
    
    # 在foldoc词典中查询bash单词的含义
    curl dict://dict.org/d:bash:foldoc
```

## Use Proxy to Download a File

**为CURL设置代理**

-x 选项可以为CURL添加代理功能

```
    curl -x proxysever.test.com:3128 http://google.co.in
```

## Send Mail using SMTP Protocol

```
    curl --mail-from blah@test.com --mail-rcpt foo@test.com smtp://mailserver.com
```

# 其他

## 保存/使用网站cookie信息

```
    # 将网站的cookies信息保存到sugarcookies文件中
    curl -D sugarcookies http://localhost/sugarcrm/index.php
    
    # 使用上次保存的cookie信息
    curl -b sugarcookies http://localhost/sugarcrm/index.php
```

## 传递请求数据

默认curl使用`GET`方式请求数据，这种方式下直接通过URL传递数据

可以通过 `--data/-d` 方式指定使用`POST`方式传递数据

```
    # GET
    curl -u username https://api.github.com/user?access_token=XXXXXXXXXX
    
    # POST
    curl -u username --data "param1=value1&param2=value" https://api.github.com
    
    # 也可以指定一个文件，将该文件中的内容当作数据传递给服务器端
    curl --data @filename https://github.api.com/authorizations
```

**注**：默认情况下，通过POST方式传递过去的数据中若有特殊字符，首先需要将特殊字符转义在传递给服务器端，如value值中包含有空格，则需要先将空格转换成%20，如：

```
    curl -d "value%201" http://hostname.com
```

在新版本的CURL中，提供了新的选项 `--data-urlencode`，通过该选项提供的参数会自动转义特殊字符。

```
    curl --data-urlencode "value 1" http://hostname.com
```

除了使用`GET`和`POST`协议外，还可以通过 `-X` 选项指定其它协议，如：

```
    curl -I -X DELETE https://api.github.cim
```

上传文件

```
    curl --form "fileupload=@filename.txt" http://hostname/resource
```

# curl网站开发指南

## 一、查看网页源码

直接在curl命令后加上网址，就可以看到网页源码。我们以网址www.sina.com为例（选择该网址，主要因为它的网页代码较短）：

```
    curl www.sina.com
```

如果要把这个网页保存下来，可以使用`-o`参数，这就相当于使用wget命令了。

```
    curl -o [文件名] www.sina.com
```

## 二、自动跳转

有的网址是自动跳转的。使用`-L`参数，curl就会跳转到新的网址。

```
    curl -L www.sina.com
```

键入上面的命令，结果就自动跳转为www.sina.com.cn。

## 三、显示头信息

`-i`参数可以显示http response的头信息，连同网页代码一起。

```
    curl -i www.sina.com
```

`-I`参数则是只显示http response的头信息。

## 四、显示通信过程

`-v`参数可以显示一次http通信的整个过程，包括端口连接和http request头信息。

```
    curl -v www.sina.com
```

如果你觉得上面的信息还不够，那么下面的命令可以查看更详细的通信过程。

```
    curl --trace output.txt www.sina.com
```

或者

```
    curl --trace-ascii output.txt www.sina.com
```

运行后，请打开output.txt文件查看。

## 五、发送表单信息

发送表单信息有`GET`和`POST`两种方法。GET方法相对简单，只要把数据附在网址后面就行。

```
    curl example.com/form.cgi?data=xxx
```

POST方法必须把数据和网址分开，curl就要用到`--data`参数。

```
    curl -X POST --data "data=xxx" example.com/form.cgi
```

如果你的数据没有经过表单编码，还可以让curl为你编码，参数是`--data-urlencode`。

```
    curl -X POST--data-urlencode "date=April 1" example.com/form.cgi
```

## 六、HTTP动词

curl默认的HTTP动词是`GET`，使用`-X`参数可以支持其他动词。

```
    curl -X POST www.example.com
```

```
    curl -X DELETE www.example.com
```

## 七、文件上传

假定文件上传的表单是下面这样：

```
    <form method="POST" enctype='multipart/form-data' action="upload.cgi">
    　　　　<input type=file name=upload>
    　　　　<input type=submit name=press value="OK">
    </form>
```

你可以用curl这样上传文件：

```
    curl --form upload=@localfilename --form press=OK [URL]
```

## 八、Referer字段

有时你需要在`http request`头信息中，提供一个`referer`字段，表示你是从哪里跳转过来的。

```
    curl --referer http://www.example.com http://www.example.com
```

## 九、User Agent字段

这个字段是用来表示客户端的设备信息。服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版。

iPhone4的User Agent是

```
    Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_0 like Mac OS X; en-us) AppleWebKit/532.9 (KHTML, like Gecko) Version/4.0.5 Mobile/8A293 Safari/6531.22.7
```

curl可以这样模拟：

```
    curl --user-agent "[User Agent]" [URL]
```

## 十、cookie

使用`--cookie`参数，可以让curl发送`cookie`。

```
    curl --cookie "name=xxx" www.example.com
```

至于具体的cookie的值，可以从`http response`头信息的`Set-Cookie`字段中得到。

`-c cookie-file`可以保存服务器返回的cookie到文件，`-b cookie-file`可以使用这个文件作为cookie信息，进行后续的请求。

```
    　curl -c cookies http://example.com
    　curl -b cookies http://example.com
```

## 十一、增加头信息

有时需要在`http request`之中，自行增加一个头信息。`--header`参数就可以起到这个作用。

```
    curl --header "Content-Type:application/json" http://example.com
```

## 十二、HTTP认证

有些网域需要HTTP认证，这时curl需要用到`--user`参数。

```
    curl --user name:password example.com
```
    
# 参考

- [curl网站开发指南](http://www.ruanyifeng.com/blog/2011/09/curl.html)
- [15 Practical Linux cURL Command Examples](https://www.thegeekstuff.com/2012/04/curl-examples/)
- [CURL常用命令](https://www.cnblogs.com/gbyukg/p/3326825.html)
- [Using curl to automate HTTP jobs](https://curl.haxx.se/docs/httpscripting.html)
- [CURL 百度百科](https://baike.baidu.com/item/curl/10098606?fr=aladdin)
- [Linux curl命令详解](http://blog.csdn.net/wh211212/article/details/54285921)
- [command line tool and library for transferring data with URLs](https://curl.haxx.se/)
