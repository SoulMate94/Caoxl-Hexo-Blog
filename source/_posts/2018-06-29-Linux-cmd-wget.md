---
title: Linux 命令 「wget」
date: 2018-06-29 15:22:16
categories: Linux cmd
tags: [Linux]
---

> **wget命令**用来从指定的`URL`下载文件。

<!-- more -->

`wget`非常稳定，它在带宽很窄的情况下和不稳定网络中有很强的适应性，
如果是由于网络的原因下载失败，`wget`会不断的尝试，直到整个文件下载完毕。
如果是服务器打断下载过程，它会再次联到服务器上从停止的地方继续下载。这对从那些限定了链接时间的服务器上下载大文件非常有用。

# 命令选项

- `-a`<日志文件>: 在指定的日志文件中记录资料的执行过程
- `-A`<后缀名>:  指定要下载文件的后缀名, 多个后缀名之间使用逗号进行分隔
- `-b`:   进行后台的方式运行`wget`
- `-B`<连接地址>: 设置参考的连接地址的基地地址
- `-c`:   继续执行上次终端的任务
- `-C`<标志>:   设置服务器数据块功能标志`on`为激活,`off`为关闭,默认值为`on`
- `-d`:   调试模式运行指令
- `-D`<域名列表>: 设置顺着的域名列表, 域名之间用","分隔
- `-e`<指令>:   作为文件"/wgetrc"中的一部分执行指定的指令
- `-h`:   显示指令帮助信息
- `-i`<文件>:   从指定文件获取要下载的`URL`地址
- `-l`<目录列表>: 设置顺着的目录列表,多个目录用","分隔
- `-L`:   仅顺着关联的连接
- `-r`:   递归下载方式
- `-nc`:  文件存在时,下载文件不覆盖原有文件
- `-nv`:  下载时只显示更新和出错信息,不显示指令的详细执行过程
- `-q`:   不显示指令执行过程
- `-nh`:  不查询主机名称
- `-v`:   显示详细执行过程
- `-V`:   显示版本信息
- `--passive-ftp`:    使用被动模式`PASV`连接`FTP`服务器
- `--follow-ftp`:     从`HTML`文件中下载`FTP`连接文件

# 命令参数

- `URL`:  下载指定的URL地址


# 实例

- **使用`wget`下载单个文件**

```
    wget http://linuxde.net/testfile.zip
```

- **下载并以不同的文件名保存**

```
    wget -O wordpress.zip http://www.linuxde.net/download/aspx?id=1080
```

`wget`默认会以最后一个符合/的后面的字符来命令，对于动态链接的下载通常文件名会不正确。

- **wget限速下载**

```
    wget --limit-rate=300k http://www.linuxde.net/testfile.zip
```

当你执行wget的时候，它默认会占用全部可能的宽带下载。但是当你准备下载一个大文件，而你还需要下载其它文件时就有必要限速了

- **使用`wget`断点续传**

```
    wget -c http://www.linuxde.net/testfile.zip
```

使用`wget -c`重新启动下载中断的文件，对于我们下载大文件时突然由于网络等原因中断非常有帮助，我们可以继续接着下载而不是重新下载一个文件。
需要继续中断的下载时可以使用`-c`参数。


- **使用wget后台下载**

```
    wget -b http://www.linuxde.net/testfile.zip
```

对于下载非常大的文件的时候，我们可以使用参数`-b`进行后台下载，你可以使用以下命令来察看下载进度：

```
    tail -f wget-log
```

- **伪装代理名称下载**

```
    wget --user-agent="Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.16 (KHTML, like Gecko) Chrome/10.0.648.204 Safari/534.16" http://www.linuxde.net/testfile.zip
```

有些网站能通过根据判断代理名称不是浏览器而拒绝你的下载请求。不过你可以通过`--user-agent`参数伪装。

- **测试下载链接**

当你打算进行定时下载，你应该在预定时间测试下载链接是否有效。我们可以增加`--spider`参数进行检查。

```
    wget --spider URL
```

- **增加重试次数**

```
    wget --tries=40 URL
```

如果网络有问题或下载一个大文件也有可能失败。`wget`默认重试`20`次连接下载文件。如果需要，你可以使用`--tries`增加重试次数。


- **下载多个文件**

```
    wget -i filelist.txt
```

首先，保存一份下载链接文件：

```
    cat > filelist.txt
    url1
    url2
    url3
    url4
```

接着使用这个文件和参数`-i`下载。

- **镜像网站**

```
    wget --mirror -p --convert-links -P ./LOCAL URL
```

接着使用这个文件和参数`-i`下载。

- `--mirror`:   开户镜像下载
- `-p`: 下载所有为了`html`页面显示正常的文件
- `--convert-links`: 下载后,转换成本地的链接
- `-P ./LOCAL`: 保存所有文件和目录到本地指定目录


- **过滤指定格式下载**

```
    wget --reject=gif ur
```

下载一个网站，但你不希望下载图片，可以使用这条命令。

- **把下载信息存入日志文件**

```
    wget -o download.log URL
```

不希望下载信息直接显示在终端而是在一个日志文件，可以使用。

- **限制总下载文件大小**

```
    wget -Q5m -i filelist.txt
```

当你想要下载的文件超过`5M`而退出下载，你可以使用。注意：这个参数对单个文件下载不起作用，只能递归下载时才有效。

- **下载指定格式文件**

```
    wget -r -A.pdf url
```

可以在以下情况使用该功能：

- 下载一个网站的所有图片。
- 下载一个网站的所有视频。
- 下载一个网站的所有`PDF`文件。

- **FTP下载**

```
    wget ftp-url
    wget --ftp-user=USERNAME --ftp-password=PASSWORD url
```

可以使用`wget`来完成`ftp`链接的下载。

- **使用`wget`匿名`ftp`下载**

```
    wget ftp-url
```

- **使用`wget`用户名和密码认证的`ftp`下载**

```
    wget --ftp-user=USERNAME --ftp-password=PASSWORD url
```
