---
title: Linux 管理总结
date: 2018-08-13 09:29:51
categories: Linux
tags: [Linux, 服务器]
---

> 总结一下平日在 `GNU/Linux`，没有特殊说明都是在 `CentOS` 上经常使用到的命令／工具的使用，但有些命令同样适用于 `macOS`。
> > 好记性不如烂笔头, 现在应该没有哪个公司是断网开发吧.

<!--more-->

# SSH/SCP/SFTP

## SSH管理

```
    # 登录远程主机
    ssh user@domain-name
    ssh <domain-alias>    # 在 ~/.ssh/config 中配置
    
    ###### 配置 SSH Key 实现免密码远程操作 [参考]
    # 本地配置
    cd ~/.ssh
    ssh-keygen -t rsa -C 'USER@domain-name' -f id_rsa_xxx
    
    ssh-copy-id -i ~/.ssh/id_rsa_xxx.pub www.remote-host.com    # will ask for password if you are the first time to add public key
    
    # Or: scp id_rsa_xxx.pub USER@domain-name:/USER/.ssh/authorized_keys
    # Or: cat ~/.ssh/id_rsa.pub | ssh user@12.34.56.78 "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"
    
    
    vim config
    
      Host domain-alias
            HostName domain-name
            User USER
            Port 22
            IdentityFile ~/.ssh/id_rsa_xxx
    
    # 远程配置
    cd ~/.ssh
    chown -R user:user ../.ssh
    chmod 0500 ../.ssh
    chmod 0400 ./*
```

### ssh-add

- 添加记住 `passphrase`

如果生存公钥对的时候使用了 `passphrase` 可以使用 `ssh-add` 记住该公钥对应私钥的密码，避免每次登录都要输入使用秘钥的密码。

```
    ssh-add -K /path/to/ras_file
    ssh-add -l
```

- 删除已记住的 `passphrase`

反之，从某个代理机器上删除记住过的私钥：

```
    # 删除单个秘钥
    ssh-add -d /path/to/ras_file
    
    # 删除所有秘钥
    add-add -D
    
    ssh-add -L
```

- 锁定与解锁秘钥

```
    ssh-add -x    # 输入两次密码锁定
    ssh-add -X    # 输入一次密码解锁
```

## SCP 上传/下载

```
    # 上传文件
    scp [-P {port}] filename user@domain-name:/path/to/save
    
    # 上传目录
    scp -r dirname user@domain-name:/path/to/save
    
    # 下载文件
    scp user@domain-name:/path/to/filename /path/to/save
    
    # 下载目录
    scp -r user@domain-name:/path/to/dirname /path/to/save
```

## SFTP 上传/下载

```
    # 1. 连接远程服务器
    sftp user@domain-name
    
    # 2. 下载文件／目录
    get /path/to/filename /path/to/save
    get -r /path/to/dirname /path/to/save
    
    # 3. 上传文件／目录
    put /path/to/filename /path/to/save
    put -r /path/to/dirname /path/to/save
    
    # 4. 查询本机当前所在路径
    lpwd
    
    # 5. 查询远程主机当前所在路径
    pwd
    
    # 6. 改变本地路径
    lcd /path/to
    
    # 7. 改变远程路径
    cd /path/to
    
    # ... 更多 shell 指令规律类似: 
    # 操作远程用原生 Shell 指令
    # 操作本地用 `l` + 原生 Shell 指令
    
    # 8. 退出 sftp
    exit | quit | bye
```

**说明:**

- `scp 源 目标` ，因此 scp 既可以下载又可以上传，只需要调整下参数顺序，因为文件／目录都是：从 源 到 目标。
- 上传下载目录时，最好先压缩成单个文件再按文件的方式进行传输。
- 若指定的路径不存在将自动创建。
- 上面的 `domain-name` 都可以用相应的 IP 地址来代替。
- 若已经配置过 `SSH Key`，则在使用上述命令的时候便不再需要每次都输入密码。
- SFTP 中，`get` 和 `put` 若不指定目标保存路径，则默认为远程主机／本机当前所在路径。

# rsync

## 同步本地和远程代码

```
    # 手动同步
    rsync -avz --progress --delete --no-o --no-g -m --chmod=Du=rwx,Dgo=rx,Fu=rw,Fog=r  /env/vagrant/www/app.dev/ root@app.dev:/data/wwwroot/app.dev
```

> 路径需要修改成你自己的


# screen/tmux

```
    # screen
    screen    # start a new screen session
    screen -S <session_id>    # start a new screen session with name
    screen -ls
    screen -r <session_id>       # recover a screen session detached
    screen -D -r <session_id>    # recover a screen session even if attached
    screen -X -S [session #id you want to kill] quit|exit
    
    Ctrl + A + D => 退出当前 Session
    
    # tmux
    tmux    # start a new tmux session
    tmux ls
    tmux new -s <session_id>
    tmux at/attach -t <session_id>
    ## rename a session
    ctrl+b => `:` => rename-session [-t current-name] [new-name] (第二可选参数不填则重命名当前 attach 的会话)
    
    Ctrl + B + D => 退出当前 Session
    Ctrl + B + " => 水平分屏
    Ctrl + B + % => 垂直分屏
    Ctrl + B + 上／下／左／右 => 分屏间切换
```

# 解压缩

- `.xz`

```
    xz -d xxx.tar.xz    # x
```

- `.tar`

```
    tar -xvf  xxx.tar           # x
    tar -cf all.tar *.jpg       # -c 表示产生新的包
    tar -rf all.tar *.gif       # -r 表示增加文件
    tar -uf all.tar logo.gif    # -u 表示更新文件
    tar -tf all.tar             # -t 列出文件
```

- `gzip`

```
    gzip /path/to/file_or_folder
    
    gzip -rv /path/to/file_or_folder
    gzip -l /path/to/file_or_folder
    gzip -dv /path/to/file_or_folder
    gzip -dr /path/to/file_or_folder
    
    # Example:
    gzip -d 1.sql.gz | mysql -h127.0.0.1 -u root -p db_demo
```

- `.tar.gz`

```
    tar -xzvf xxx.tar.gz         # x
    tar -czf all.tar.gz *.jpg    # tar 调用 gzip
```

- `.tar.bz2`

```
    tar -xjvf xxx.tar.bz2        # x
    tar -cjf all.tar.bz2 *.jpg   # tar 调用 bzip2
```

- `.tar.Z`

```
    tar –xZvf xxx.tar.Z         # x
    tar -cZf all.tar.Z *.jpg    # tar 调用 compress
```

- `.rar`

```
    unrar e xxx.rar    # x
```

- `.zip`

```
    unzip xxx.zip                  # x
    zip -r xxx.zip /path/to/xxx    # c
```

# 定时任务

## `atd`

```
    echo 'schedule job' | at now
```

## `Cron`/`Crontab`

- 命令格式：`分 时 日 月 周` 命令，比如：

```
    # 0～59 0～23 1～31 1～12 0～6 command
    */1 * * * * php /data/wwwroot/default/cron.php    # 每分钟执行一次 cron.php
    0 8 * * * /sbin/service/sshd start                # 每天 8:00 开启 ssh 服务
```

其中，`*` 代表取值范围内的数字，`/` 代表「每」。

### 未预装 `crond CentOS` 下安装 `crond` 服务

```
    # CentOS 6
    yum install vixie-cron
    # yum install crontabs     # install cronie,cronie-anacron,exim the same time
    
    service crond start
    chkconfig crond on
    
    # CentOS 7
    yum install cronie
```

这里有个不错的在线校准网站：<u>https://crontab.guru/</u>。

- 其他命令

```
    crontab -e    # 编辑当前用户的 cron 服务
    crontab -l    # 列出当前用户的 cron 服务的详细内容
    crontab -r    # 删除某个用户的所有 cron 服务
    service crond start|restart|stop    # 开启／重启／停止 crond 服务
```

# 网络工具

## `iptables`

- 查看 `iptables` 防火墙配置信息

```
    /etc/init.d/iptables status
```

- 放行某个端口

```
    # 如：放行 3306 端口
    /sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
```

- 保存配置信息

```
     # !!! 在修改防火墙规则后需要保存配置才能生效
    service iptables save
```

- 打开、重启、关闭 `iptables` 服务

```
    service iptables restart|start|stop
    # 或者
    /etc/init.d/iptables restart|start|stop
```

- 取消 `iptables` 服务自启动

```
    # 相当于永久关闭 iptables
    chkconfig –level 35 iptables off
```

## `nmap`

```
    yum install -y nmap
    
    # 手册
    info nmap
    
    # 查看本机当前开放的端口
    nmap -sTU -O localhost
    
    # 检查 本机所在网段有多少台 live 机器
    nmap -sP 192.168.32.0/24
```

## `netstat`

从 `netstat` 里可以看到自己机器正在监听的端口、相关程序以及当前的连接数、连接来自何方等数据，然后有针对性的进行关闭相关服务或者用防火墙来屏蔽、过滤对本机服务的访问。

```
    netstat -antp       # 所有连接、数字显示主机、端口、TCP 连接、监听的程序
    netstat -anup       # 所有连接、数字显示主机、端口、UDP 连接、监听的程序
    netstat -s          # 统计所有（开机至今的）连接数据，包括 tcp、udp 等
    netstat -st         # 统计所有 tcp 连接数据
    netstat -su         # 统计所有 udp 连接数据
```

## `lsof`

查看哪个程序监听着哪个端口。

```
    lsof -i:80
```

## `dig`

```
    [root@caoxl ~]# dig github.com
    
    # 查询参数和统计
    ; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> github.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58065
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0
    
    # 查询内容 (A 代表 address)
    ;; QUESTION SECTION:
    ;github.com.			IN	A
    
    # DNS 服务器的答复 (31 代表缓存时间, 即TTL, 表示31秒内不用查询查询
    ;; ANSWER SECTION:
    github.com.		31	IN	A	192.30.253.112
    github.com.		31	IN	A	192.30.253.113
    
    # DNS 服务器的一些传输信息
    ;; Query time: 0 msec
    ;; SERVER: 100.100.2.136#53(100.100.2.136)
    ;; WHEN: Mon Aug 13 10:05:46 CST 2018
    ;; MSG SIZE  rcvd: 60
```

如果有 `AUTHORITY SECTION:` 段，则为 dig 域名的 NS 记录（Name Server），即哪些服务器负责管理所查询域名的DNS记录。向任何一台 NS 查询都能查到所查域名的 IP 信息。

此时，会一同返回 `ADDITIONAL SECTION:` 段，即为 NS 的 IP。

```
    # 精简查询
    dig +short github.com
    
    # 显示 DNS 的整个分级查询过程
    dig +trace github.com
    
    # 指定 DNS 服务器查询
    dig @8.8.8.8 github.com
    
    # 单独查看每一级域名的 NS 记录
    dig ns com
    dig ns github.com
    
    # 查询 PTR 记录（从 IP 地址查询域名）
    dig -x 1.2.3.4
    
    # 查询指定的记录类型
    dig a github.com
    dig mx github.com
```

## `host`

简化版 dig。

```
    [root@caoxl ~]# host github.com
    github.com has address 192.30.253.113
    github.com has address 192.30.253.112
    github.com mail is handled by 10 alt4.aspmx.l.google.com.
    github.com mail is handled by 1 aspmx.l.google.com.
    github.com mail is handled by 5 alt1.aspmx.l.google.com.
    github.com mail is handled by 10 alt3.aspmx.l.google.com.
    github.com mail is handled by 5 alt2.aspmx.l.google.com.
    
    [root@caoxl ~]# host 192.30.253.113
    113.253.30.192.in-addr.arpa domain name pointer lb-192-30-253-113-iad.github.com.
```

## `nslookup`

互动式地查询域名记录。

```
    [root@caoxl ~]# nslookup
    > caoxl.com
    Server:		100.100.2.136
    Address:	100.100.2.136#53
    
    Non-authoritative answer:
    Name:	caoxl.com
    Address: 47.91.221.85
```

## `whois`

查看域名的注册情况。

```
    whois github.com
```

## `wget`

- 递归下载整站

```
    wget -r -p -np -k http://www.example.com
    
    wget \
         --recursive # download the entire Web site \
         --no-clobber # don't overwrite any existing files (used in case the download is interrupted and resumed) \
         --page-requisites # get all the elements that compose the page (images, CSS and so on) \
         --html-extension # save files with the .html extension \
         --convert-links #convert links so that they work locally, off-line \
         --restrict-file-names=windows # modify filenames so that they will work in Windows as well \
         --domains cloud.google.com # don't follow links outside cloud.google.com \
         --no-parent https://cloud.google.com/apis/design/resources # don't follow links outside the directory api/design/resources
    
    # 实例：下载在线电子书网站到本地
    wget --mirror --convert-links --no-parent --no-verbose https://landing.google.com/sre/book/
```

## `curl`

- `RESTFul` 请求

```
    curl -X GET http://www.example.com
```

- 打印返回请求头

```
    curl -I http://www.example.com
    curl --head http://www.example.com
```

- 自定义头信息

```
    curl -i
    -X POST http://www.example.com/api/with/key \
    -H "Accept: application/json" \
```

- `POST JSON`

```
    curl -X POST http://www.example.com/api/with/key \
    -H "Accept: application/json" \
    -d '{"username":"xyz","password":"xyz"}'
```

- 输出请求详情

```
    curl -v https://www.google.com
```

- 文件内容作为请求 `payload`

```
    curl -vX POST http://example.com \
    -H 'Content-Type: application/json; charset=utf-8'
    -d @/path/to/file.json
```

- `HTTP base authorization`

```
    curl -u user:passwd http://example.com
    
    # Or:
    curl http://user:passwd@example.com
```

# 进程管理

## `supervisor`

```
    yum info supervisor
    yum install -y supervisor
    
    service supervisord start|stop|restart
    
    supervisorctl reread
    supervisorctl start proj-worker:*
    
    vim /etc/supervisord.conf
    # 以编辑 Laravel 项目队列后台进程管理为例
    [program:proj-name]
    process_name=%(program_name)s_%(process_num)02d
    command=php /data/wwwroot/www.proj.com/artisan queue:work database --queue=phjs-cc-%(process_num)2d --memory=256 --sleep=2 --tries=2 --daemon
    autostart=true
    autorestart=true
    user=www
    numprocs=16
    redirect_stderr=true
    stdout_logfile=/data/wwwroot/www.proj.com/storage/logs/sv-worker.log
```

### `“unix:///tmp/supervisor.sock no such file”`

在执行 `supervisorctl -c /etc/supervisord.conf` 的时候，出现该报错。

原因是：

> `supervisor`默认配置会把`socket`文件和`pid`守护进程生成在`/tmp/`目录下，`/tmp/`目录是缓存目录，Linux会根据不同情况自动删除其下面的文件。

解决办法：

```
    vi /etc/supervisord.conf
    
    `
    [unix_http_server]
    file=/var/run/supervisor.sock
    
    [supervisord]
    logfile=/var/log/supervisor/supervisord.log
    
    [supervisorctl]
    serverurl=unix:///var/run/supervisor.sock   # 必须和 `unix_http_server` 一一对应
    `
    
    supervisorctl update
    # supervisord -c /etc/supervisord.conf
```

## `systemd` 守护进程

创建一个自定义守护进程服务配置文件：

```
    vi /lib/systemd/system/phpd-test.service
    
    `
    [Unit]
    Description=PHP Daemon Demo
    
    [Service]
    User=phpdemo
    Group=phpdemo
    WorkingDirectory=/home/phpdemo
    Restart=always
    ExecStart=/usr/bin/php phpd-test.php
    `
    
    systemctl status phpd-test    # systemd does not start new services automatically
    systemctl start mydaemon
```

测试：`kill phpd-test` 相关进程，查看 `systemd` 是否自动重启。

### 相关路径

- services: `/lib/systemd/system/*.service`
- logs: `/var/log/syslog`

> 日志文件可以通过 journalctl —since "2018-08-13 12:13:31" 来快速获取

### 参考

- <u>http://rustamagasanov.com/blog/2017/02/24/systemd-example-for-a-simple-ruby-daemon-supervision/</u>
- <u>
https://serversforhackers.com/c/process-monitoring-with-systemd</u>

## 批量杀死进程

```
    ps -ef | grep php | grep -v grep | awk '{print $2}' | xargs kill -9
```

## 后台运行

```
    nohup ~/shell/server-git-sync > /dev/null 2>&1 &
```

## 查看日志文件变动

```
    tail -f /tmp/git-sync-client.log
```

# 文件系统管理

## 批量重命名

```
    i=0; for f in *.mp4 ; do let i=i+1; mv "$f" $i".mp4" ; done
```

## 递归创建文件夹

```
    mkdir -p root/{com,net,org}/{taobao,tmall}/{www,detail}
```
 
## 将文件隐藏到图片

```
    copy*/b*d:1.jpg*+*d:2.rar*d:3.jpg
```
 
# 常用命令

## `man`

查看命令的使用说明。

```
    man htop
```

## `cheat`

查看某个常用命令的使用举例。

```
    yum install docopt
    yum install python-pip
    git clone git@github.com:chrisallenlane/cheat.git
    cd cheat/
    python setup.py install
    
    # Check
    cheat -v
    
    # Usage demo
    cheat tar
    cheat curl
```

## `find`

```
    # 找到文件大小小于 10000k 的文件并删除
    find . -size -10000k | xargs rm -rf
    
    # 找到文件名中包含字符串的文件
    find . -name '*linux*'    # 区分大小写
    
    find . -iname '*linux*'    # 不区分大小写
    
    # 找到后缀为 jpg/JPG 的文件
    find . -iname '*.jpg'
    
    # 找出目录类型
    find . -type d
    
    # 找出文件类型
    find . -type f
    
    # 找到八进制权限为 777 的文件
    find . -type f -perm 777
    
    # 找到文件名以 .tmp 结尾的并删除
    find . -name '*.txt' -exec rm '{}' \;
    
    # 删除所有空文件夹
    find . -type d -empty -exec rmdir {} \;
    
    # 找到所有最后修改时间为 7 天前的文件
    find . -type f -mtime +7d -ls
    
    # 从制定文件类型中找到查找字符串
    find . -name '*.php' | xargs grep -ri 'eval'
    
    # 找到文件大于 1M 的并排序
    find . -size +1M -type f -print0 | xargs -0 ls -Ssh | sort -z
    
    # 找到所有名称为 logs 的所有目录 最大深度为 2
    find . -maxdepth 2 -name logs -type d
    
    # 找到所有不在 .git 目录中的文件
    find . ! -iwholename '*.git*' -type f
    
    # 找到当前目录下所有文件并修改其权限
    find . -type f -exec chmod 644 {} \;
```

## `du`

```
    # 统计当前目录大写 输出目录深度为 1
    du . -h --max-depth=1
```

# VIM

## 快捷键

- 上翻整页：`ctrl + f`
- 上翻半页：`ctrl + u`
- 下翻整页：`ctrl + b`
- 下翻半页：`ctrl + d`
- 左缩进：命令模式下按 V，然后 << (shift + <)
- 右缩进：命令模式下按 V，然后 >>

## 常用命令

```
    # 显示行号
    set nu
    # 关闭行号
    set nu!
    set nonu
    # 设置缩进
    set shiftwidth=4
    # 设置Tab宽度
    set tabstop=4
```

# 用户管理

## 查看并剔除用户

```
    # 查看用户
    w
    who
    whoami
    who am i
    
    [root@caoxl ~]# w
     10:48:34 up 10 days,  1:09,  1 user,  load average: 0.00, 0.01, 0.05
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    root     pts/0    61.140.74.36     10:00    2.00s  0.04s  0.00s w
    [root@caoxl ~]# who
    root     pts/0        2018-08-13 10:00 (61.140.74.36)
    [root@caoxl ~]# whoami
    root
    [root@caoxl ~]# who am i
    root     pts/0        2018-08-13 10:00 (61.140.74.36)
    
    # 剔除指定终端的用户
    pkill -kill -t pts/2
```

# FAQ

- `scp`**not a regular file?**

```
    # 带 `-r` 选项
    scp -r xxx/
```

- **Linux 启动文件系统检查失败：Inode 2891983, end of extent exceeds allowed value**

```
    # 1. 查看 /etc/fstab 文件中是否写入了错误的文件系统或者磁盘的分区信息
    # 1.1 定位／打印区块设备属性
    blkid
    
    # 1.2 查看系统文件系统的固定信息和 blkid 是否一致
    vim /etc/fstab
    
    # 2. 修复分区
    e2fsck -C0 -p -f -v /dev/sda1
```

- **SSH: connection rest by peer**

> http://stackoverflow.com/questions/1434451/what-does-connection-reset-by-peer-mean

# 参考

- [Linux 管理个人总结](http://blog.cjli.info/2016/04/19/Linux-Management-Summaries/)
- [How To Set Up SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
- [云服务器 ECS Linux 系统 /etc/fstab 错误配置导致系统启动异常](https://help.aliyun.com/knowledge_detail/41460.html)
- [DNS 原理入门](http://www.ruanyifeng.com/blog/2016/06/dns.html)
- [Linux 技巧：让进程在后台可靠运行的几种方法](https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/)
- [Stupid ssh-add Tricks](https://stuff-things.net/2016/02/11/stupid-ssh-add-tricks/)