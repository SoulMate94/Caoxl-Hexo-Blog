---
title: Linux 命令 「好玩的」
date: 2018-06-06 16:19:52
categories: Linux cmd
tags: [Linux]
---

# 好玩的 `Linux` 命令

持续更新~~~

<!-- more -->

# `wall`

**wall命令**用于向系统当前所有打开的终端上输出信息。类似于广播.

```
    [root@iZ58vt8oll8aheZ /]# wall hello
    [root@iZ58vt8oll8aheZ /]# 
    Broadcast message from root@iZ58vt8oll8aheZ (pts/0) (Wed Jun  6 16:18:37 2018):
    
    hello
```

# `mesg`

**mesg命令**用于设置当前终端的写权限，即是否让其他用户向本终端发信息。
将`mesg`设置`y`时，其他用户可利用`write`命令将信息直接显示在您的屏幕上。

## 命令参数

- -`y/n`：y表示运行向当前终端写信息，n表示禁止向当前终端写信息。

# `man`

**man命令**是Linux下的帮助指令，通过man指令可以查看Linux中的指令帮助、配置文件帮助和编程帮助等信息。

## 命令参数

- `-a`: 在所有的`man`帮助手册中搜索
- `-f`: 等价于`whatis`指令,显示给定关键字的间断描述信息
- `-P`: 指定内容时使用分页程序
- `-M`: 指定`man`手册搜索的路径


# `whatis`

**whatis命令**是用于查询一个命令执行什么功能，并将查询结果打印到终端上。

> whatis命令等同于使用man -f命令。

```
    [root@iZ58vt8oll8aheZ /]# whatis ls
    ls (1)               - list directory contents
    [root@iZ58vt8oll8aheZ /]# man -f ls
    ls (1)               - list directory contents
```

# `cal`

**cal命令**用于显示当前日历，或者指定日期的日历。

## 命令参数

- `-1`: 显示单月输出
- `-3`: 显示临近三个月的日历
- `-s`: 将星期日作为月的第一天
- `-m`: 将星期一作为月的第一天
- `-j`: 显示从1月1日起第多少天
- `-y`: 显示当前年的日历


# `speedtest-cli`

**speedtest-cli**是一个使用python编写的命令行脚本，通过调用speedtest.net测试上下行的接口来完成速度测试，
最后我会测试运维生存时间所在服务器的外网速度。项目地址：https://github.com/sivel/speedtest-cli

## 安装

- `pip`方式

```
    pip install speedtest－cli
```

- `easy_install`方式

```
    easy_install speedtest-cli
```

- `github＋pip`方式

```
    pip install git+https://github.com/sivel/speedtest-cli.git
    
    // 或者
    git clone https://github.com/sivel/speedtest-cli.git
    python speedtest-cli/setup.py install
```

- 下载脚本方式

```
    wget -O speedtest-cli https://raw.github.com/sivel/spe ... er/speedtest_cli.py
    chmod +x speedtest-cli
    
    // 或者
    curl -o speedtest-cli https://raw.github.com/sivel/spe ... er/speedtest_cli.py
    chmod +x speedtest-cli
```

## 实例

```
    [root@izj6c6djex81rijczh0t8yz ~]# speedtest-cli --list | grep China
    10267) Interoute VDC (Hong Kong, China) [0.00 km]
     2993) Website Solution Limited (Hong Kong, China) [0.00 km]
    12990) QTS Data Centers (Hong Kong, China) [0.00 km]
     1536) STC (Hong Kong, China) [0.00 km]
     ....
```

## 结果解释

> 6611) China Mobile,Guangdong (Guangzhou, China) [134.80 km]

- `6611`: 服务器id
- `China Mobile`: 服务器所属 (这里是中国移动)
- `Guangdong (Guangzhou, China)`: 服务器所在地址
- `[134.80 km]`: 两台服务器地理位置之间距离，我这台机器在香港，和广州相距134.80公里.

## 外网速度测试

```
    [root@izj6c6djex81rijczh0t8yz ~]# speedtest-cli --server=6611 --share
    Retrieving speedtest.net configuration...
    Testing from Alibaba (47.91.221.85)...
    Retrieving speedtest.net server list...
    Retrieving information for the selected server...
    Hosted by China Mobile,Guangdong (Guangzhou) [134.80 km]: 11.005 ms
    Testing download speed................................................................................
    Download: 86.75 Mbit/s
    Testing upload speed................................................................................................
    Upload: 1.80 Mbit/s
    Share results: http://www.speedtest.net/result/7370743213.png
```

# `lastb`

**lastb命令**用于显示用户错误的登录列表，此指令可以发现系统的登录异常

## 命令参数

- `-a`: 把从何处登入系统的主机名称或ip地址显示在最后一行
- `-d`: 将IP地址转换成主机名称
- `-f`<记录文件>: 指定记录文件
- `-n`<显示列数>或-<显示列表>: 设置列出名单的显示列数
- `-R`: 不显示登入系统的主机名称或IP地址
- `-x`: 显示系统关机,重新开机, 以及执行等级的改变等信息


# `finger`

**finger命令**用于查找并显示用户信息。包括本地与远端主机的用户皆可，帐号名称没有大小写的差别。
单独执行finger指令，它会显示本地主机现在所有的用户的登陆信息，包括帐号名称，真实姓名，登入终端机，闲置时间，登入时间以及地址和电话。

## 安装

```linux
   yum install finger 
```

## 命令参数

- `-l`: 列出该用户的账号名称, 真实姓名, 用户专属目录,登入所用Shell,
        登入时间,转信地址,电子邮件状态,还有计划文件和方案文件内容
- `-m`: 排除查找用户的真实姓名
- `-s`: 列出该用户的账号名称,真实姓名,登入终端机,闲置时间,登入时间以及地址和电话
- `-p`: 列出该用户的帐号名称，真实姓名，用户专属目录，登入所用的Shell，登入时间，转信地址，电子邮件状态，
        但不显示该用户的计划文件和方案文件内容
## 实例

```
    [root@izj6c6djex81rijczh0t8yz ~]# finger
    Login     Name       Tty      Idle  Login Time   Office     Office Phone   Host
    root      root       pts/1          Jun  8 10:04                           (61.140.75.173)
```

# `ctrlaltdel`

**ctrlaltdel命令**用来设置组合键“Ctrl+Alt+Del”的功能。

## 命令参数

- `hard`: 当按下组合键“Ctrl+Alt+Del”时，立即执行重新启动操作系统，而不是先调用sync系统调用和其他的关机标准操作。
- `soft`: 当按下组合键“Ctrl+Alt+Del”时，首先向init进程发送SIGINT（interrupt）信号。由init进程处理关机操作。

# `dig`

**dig命令**是常用的域名查询工具，可以用来测试域名系统工作是否正常。

## 安装

```
    yum install bind-utils
```

## 命令选项

- `@<服务器地址>`: 指定进行域名解析的域名服务器
- `-b<ip地址>`:  当主机具有多个IP地址,指定使用本机的哪个IP地址向域名服务器发送域名查询请求
- `-f<文件名称>`: 指定dig以批处理的方式运行,指定的文件中保存着需要批处理查询的DNS任务信息
- `-P`:   指定域名服务器所使用的端口号
- `-t<类型>`:   指定要查询的DNS数据类型
- `-x<IP地址>`: 执行逆向域名查询
- `-4`:   使用IPv4
- `-6`:   使用IPv6
- `-h`:   显示指令帮助信息

## 命令参数

- 主机: 指定要查询的域名主机
- 查询类型: 指定DNS查询的类型
- 查询类: 指定查询DNS的class
- 查询选项: 指定查询选项


# `netstat`

**netstat命令**用来打印Linux中网络系统的状态信息，可让你得知整个Linux系统的网络情况。

## 命令参数

- `-a/--all`:     显示所有连线中的Socket;
- `-A<网络类型>/--<网络类型>`: 列出该网络类型连接中的相关地址;
- `-c/--continuous`:  持续列出网络状态
- `-C/--cache`:       显示路由器配置的快取信息
- `-e/--extend`:      显示网络其他相关信息
- `-F/--fib`:         显示FIB
- `-g/--groups`:      显示多重广播功能群组组员名单
- `-h/--help`:        在线帮助
- `-i/--interfaces`:  显示网络界面信息表单
- `-l/--listening`:   显示监控中的服务器Socket;
- `-M/--masquerade`:  显示伪装的网络连线;
- `-n/--numeric`:     直接使用ip地址,而不同过域名服务器
- `-N/--netlink/--symbolic`: 显示网络硬件外围设置的服务连接名称
- `-o/-timers`:       显示计时器
- `-p/--programs`:    显示正在使用Socket的程序识别码和程序名称;
- `-r/--route`:       显示Routing Table;
- `-s/--statistice`:  显示网络工作信息统计表;
- `-u/--udp`:         显示TCP传输协议的连线状况
- `-v/--verbose`:     显示指令执行过程
- `-V/--version`:     显示版本信息
- `-w/--raw`:         显示RAW传输协议的连线状况
- `-x/--unix`:        此参数的效果和指定"-A unix"参数相同
- `--ip/--inet`:      此参数的效果和指定"-A inet"参数相同

## 实例

- 列出所有端口(包括监听和未监听的)

```
    netstat -a      #列出所有端口
    netstat -at     #列出所有tcp端口
    netstat -au     #列出所有udp端口
```

- 列出所有处于监听状态的 Sockets

```
    netstat -l      #只显示监听端口
    netstat -lt     #只列出所有监听tcp端口
    netstat -lu     #只列出所有监听udp端口
    netstat -lx     #只列出所有监听UNIX端口
```

- 显示每个些而已的统计信息

```
    netstat -s      #显示所有端口的统计信息
    netstat -st     #显示TCP端口的统计信息
    netstat -su     #显示UDP端口的统计信息
```

- 在`netstat`输出中显示PID和进程名称

```
    netstat -pt
```

- 在`netstat`输出中不显示主机,端口和用户名(host,port,user)

```
    netstat -an
```

- 持续输出`netstat`信息

```
    netstat -c      #每隔一秒输出网络信息
```

- 显示系统不支持的地址族(Address Families)

```
    netstat --verbose
```

在输出的末尾有如下信息:

```
    netstat: no support for `AF IPX' on this system.
    netstat: no support for `AF AX25' on this system.
    netstat: no support for `AF X25' on this system.
    netstat: no support for `AF NETROM' on this system.
```

- 显示核心路由信息

```
    netstat -r
```

# `which`

**which命令**用于查找并显示给定命令的绝对路径，环境变量PATH中保存了查找命令时需要遍历的目录。
which指令会在环境变量$PATH设置的目录里查找符合条件的文件。
也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

## 语法

```
    which (选项) (参数)
```

## 选项

- `-n<文件名长度>`: 制定文件名长度, 指定的长度必须大于或等于所有文件中最长的文件名
- `-p<文件名长度>`: 与`-n`参数相同,但此处的>文件名长度>包含了文件的路径
- `-w`: 指定输出时栏位的宽度
- `-V`: 显示版本信息

## 参数

指令名: 指令名列表

## 实例

- 查找文件,显示命令路径

```
   [root@izj6c6djex81rijczh0t8yz test]# which pwd
   /usr/bin/pwd
   
   [root@izj6c6djex81rijczh0t8yz test]# which adduser
   /usr/sbin/adduser  
```

**说明：** `which`是根据使用者所配置的 `PATH` 变量内的目录去搜寻可运行档的！
所以，不同的 `PATH` 配置内容所找到的命令当然不一样的！

# `tree`

**tree命令**以树状图列出目录的内容。

## 命令选项

- `-a`:   显示所有文件和目录
- `-A`:   使用ASNI绘图字符显示树状图而非ASCII字符组成
- `-C`:   在文件和目录清单加上色彩, 便于区分各种类型
- `-d`:   先是目录名称而非内容
- `-D`:   列出文件或目录的更改时间
- `-f`:   在每个文件或目录之前, 显示完整的相对路径名称
- `-F`:   在执行文件, 目录, Socket, 符号连接, 管道名称,各种加上"*","/","@","|"号
- `-i`:   不以阶梯状列出文件和目录名称
- `-I`:   不显示符号范本样式或目录名称
- `-l`:   如遇到性质为符号连接的目录,直接列出该连接所指向的原始目录
- `-n`:   不在文件和目录清单上加上色彩
- `-N`:   直接列出文件和目录名称, 包括控制字符
- `-p`:   列出权限表示
- `-q`:   用"?"号取代控制字符,列出文件和目录名称
- `-s`:   列出文件和目录大小
- `-t`:   用文件和目录的更改时间排序
- `-u`:   列出文件或目录的拥有者名称,没有对应的名称时,则显示用户识别码
- `-x`:   将范围局限在现行系统中, 若指定目录下的某些子目录,其存放于另一个文件系统上，则将该目录予以排除在寻找范围外。

## 命令参数

- `目录`：执行`tree`指令，它会列出指定目录下的所有文件，包括子目录里的文件。


# `time`

**time命令**用于统计给定命令所花费的总时间。

```
    [root@izj6c6djex81rijczh0t8yz default]# time ls
    Aliyun  images  index-back.html  index.html  phpinfo.php  Server.php  socket.html
    
    real	0m0.002s
    user	0m0.000s
    sys	    0m0.001s
```

输出的信息分别显示了该命令所花费的`real`时间、`user`时间和`sys`时间。

- `real`时间是指**挂钟时间**，

也就是命令开始执行到结束的时间。这个短时间包括其他进程所占用的时间片，和进程被阻塞时所花费的时间。

- `user`时间是指**进程花费在用户模式中的CPU时间**，

这是唯一真正用于执行进程所花费的时间，其他进程和花费阻塞状态中的时间没有计算在内。

- `sys`时间是指**花费在内核模式中的CPU时间**，

代表在内核中执系统调用所花费的时间，这也是真正由进程使用的`CPU`时间。


# `wc`

> **wc命令**用来计算数字。利用wc指令我们可以计算文件的Byte数、字数或是列数，
若不指定文件名称，或是所给予的文件名为“-”，则wc指令会从标准输入设备读取数据。

```
    wc (选项)(参数)
```

## 选项

- `-c`/`--bytes`/`--chars`:   只显示Bytes数
- `-l`/`--lines`:   只显示列数
- `-w`/`--words`:   只显示字数

## 参数

- 文件:   需要统计的文件列表