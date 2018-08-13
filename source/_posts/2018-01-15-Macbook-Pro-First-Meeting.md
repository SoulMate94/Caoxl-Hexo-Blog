---
title: MacBook Pro 初见
date: 2018-01-15 16:15:33
categories: Mac
tags: Mac
---

## MacBook 初见
> 2018-1-8 14:39PM 第一次看到属于我自己的MacBook Pro

## EI Capitan 系统设置

### 触控板和手势
 - **单指点按选中：** 系统偏好设置 => 触控板 => 光标与点按 => 轻点来点按。相当于单击鼠标左键打开应用。
 - **单指锁定：** 系统偏好设置 => 辅助功能 => 鼠标与触控板 => 触控板选项 => 启用拖移 => 使用拖移锁定。点一下窗口后即可选中开始拖动
 - **双指右侧滑动**：左右滑动分别开关通知栏。
 - **四指上下：** 显示当前打开的窗口＋多桌面。
 - **四指抓取：** 打开 Launchpad，退出释放四指。
 - **双指选中：** 相当于鼠标右键属性。
 - **左右滑动：** 在浏览器历史纪录堆栈中前进后退。
 
 
 <!--more-->
 
 
### Dock
 - **Dock 居左：** 系统偏好设置 => Dock => 置于屏幕上的位置 => 左边。
 - **Dock 缩小：** 系统偏好设置 => Dock => 大小 => 向左滑动。
 - **Dock自动隐藏：** Command + Alt + D。
 - **全屏触发：** 当鼠标向 Dock 所在方向滑动时自动弹出，同顶部菜单。
 - **禁用与恢复:** Dashboard：
 
 ```
    # 禁用仪表盘
    defaults write com.apple.dashboard mcx-disabled -boolean YES && killall Dock
    ＃ 恢复仪表盘
    defaults write com.apple.dashboard mcx-disabled -boolean NO && killall Dock
 ```
### Mac OS X 键盘符号解释
    ⇧ = shift
    
    ⌃ = control
    
    ⌥ = option / alt
    
    Home = fn + ◄
    
    End = fn + ►
    
    Page Up = fn + ▲
    
    Page Down = fn + ▼
    
### 快捷键
 - **删除文件：** Command + Delete（也是删除光标前整行内容）；
 - **强制删除：** Command + Alt + Delete。
 - **删除光标前一个单词：** Option + Delete。
 - **删除光标后一个单词：** fn + Command + delete。
 - **删除光标后一个字符：** fn + delete。
 - **关闭窗口：** Command + W。
 - **查看所需单词定义：** Control＋Command ＋D。
 - **退出程序：** Command +Q；Command + Alt + Esc（强制退出）。
 - **F11：** 显示桌面（启用标准功能键后），同四指张开。
 - **连续两次 Command：** QQ Swiftly 搜索，注意是在英文输入法的前提下。
 - **最小化窗口：** Command ＋ M；Command ＋空格。
 - **关闭已最小化的窗口：Command ＋H。
 - **预览文件：** 空格键。
 - **新建文件夹：** Command + Shift＋N。
 - **打开新窗口：** 选中应用程序后按 Command ＋ N，比如 QQ。
 - **删除项目到废纸篓：** Command + Delete。
 - **清空废纸篓：** Command + Shift + Delete。
 - **截图保存整个屏幕到桌面：** shift + command + 3。
 - **保存整个屏幕到剪贴板：** control + shift + command + 3。
 - **截取指定屏幕区域到桌面：** shift + command + 4。
 - **保存指定屏幕区域到剪贴板：** control + shift + command + 4。
 - **新建程序：** command + n【比如同时登陆2个QQ】。
 - **输入法切换: ** command + space。
 - **切换窗口：** control + F4 或者command + tab。
 - **最小化当前窗口：** Command + m.
 - **最小化所有窗口：** Option + click minimize button。
 - **适合屏幕：** Option + click zoom button【在mac下没有窗口的最大化，而是适合屏幕】。
 - **关闭当前窗口：** Command + w。
 - **关闭所有窗口：** Option + click close button【比如说你打开了多个safari网页，这时想要关闭全部窗口，可以右击safari，然后退出；也可以按住option，然后点任意一个窗口的关闭按钮】
 - **窗口的移动：** 需要3指触控。第一，用中指和无名指作为一指点窗口的最顶端（类似windows标题栏），并按住不放；第二，用食指拖拽到目的位置；第三释放3指。【lion测试，小白们设置一下触控板，自己研究一下，esay】
 - **选择相邻的图标 (列表显示)：** Shift + click。
 - **选择不相邻的图标 (列表显示)：** Command + click。
 - **重命名：** 先选中，然后点 Return（enter）。
 - **文件或文件夹复制：** command + c 代表复制。 但是，注意！没有剪切的快捷键，要实现剪切功能，需要command和拖拽配合完成。具体操作：首先，选中文件，其次按住command，再次拖拽文件到目的位置后松手，最后，释放command。
 - **原位复制：** Command + d【mac下对于文件的复制不同于windows，与windows的复制相同功能的在mac下是拷贝】。
 - **创建替身(拖拉方式)：** Command + Option + 拖拉
 - **创建替身(命令方式)：** Command + l (L)【mac下的替身类似于windows下的快捷图标，但比其更富有人性化】。
 - **强制退出应用程序：** Command + Option + Escape。
 - **强制重新启动：** Command + Ctrl + 电源 key
 - **注销：** Command + Shift + q。
 - **重置 Parameter RAM：** Command + Option + p + r。
 - **更多快捷键：** Linux 很多按键操作 Mac 也适用，比如 ctrl+A（跳行首） 和 ctrl+E（跳行尾）等；把 Command 当作 Windows 的 Ctrl 或 Win，发现很多都是相同的。更多快捷键相见附录。
 
## 日常管理
 - 清理 DNS 缓存
 ```
    sudo lookupd -flushcache    # Tiger 或更低版本 Mac OS X
    sudo dscacheutil -flushcache    # Leopard 和 Snow Leopard
    sudo killall -HUP mDNSResponder   # Lion、Mountain Lion 和 Mavericks(OS X v10.10.4 or later)
    sudo discoveryutil mdnsflushcache.   # v10.10 through v10.10.3
    say cache flushed
 ```
 
 - 终端邮件
 ```
    $ mail
    > delete *
    > q
 ```
 
 - 硬盘测速
 ```
    time dd if=/dev/zero bs=1024k of=tstfile count=1024
    dd if=tstfile bs=1024k of=/dev/null count=1024
 ```
 
 - Mac OS X 查看磁盘空间
 ```
    sudo du / -h -d 1
 ```
 
 - JPEG／PNG图片压缩命令行工具
 ```
    http://blog.topspeedsnail.com/archives/2737
 ```
 
 - 彻底卸载 Mac App
 ```
    # 查找
    mdfind -name "APP_NAME"
    # 更改权限
    # 删除
    sudo rm -rf filename
 ```
 
 - 查看CPU-内存-耗电情况等终端工具
 ```
    # 实际命令以具体环境为准
    brew install htop    # CPU 内存 进程
    htop
    pip3 install trackmac    # 耗电应用统计
    tm list -m
 ```
 
 - 粘贴到剪切板
 ```
    # Windows
    clip < text.txt
    # Mac
    pbcopy < text.txt
    # Linux
    xclip
 ```
 
## Homebrew/Cask - MBP Dev 的开始

简而言之，brew 类似于 yum 和 apt-get，都是平台下的软件包管理工具，强烈推荐优先使用 homebrew 在 Mac OS X 上搭建开发环境，自带的一些开发环境虽然也可以用但是不如 brew 专业，正所谓专业的人做专业的事，为了您的宝贵时间，so。

下载安装 Homebrew：

```
    ＃ Mac OS X 内置 ruby 可直接使用 curl 下载后安装 
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

如果没有安装过 Command Line Tools，安装过程中会下载 Xcode 。
```
    brew doctor    # 自检，成功则提示 "Your system is ready to brew."
```

除了brew，Mac OS X 上还有一个更适合开发者的软件管理工具：cask。它的优点是：软件支持更全面＋更新速度快＋速度比App Store快。
```
    brew tap phinze/homebrew-cask && brew install brew-cask
```

好了，以后安装编程环境和开发工具什么的就 brew install 起来吧。

下面的软件都可以通过 brew 来安装。

### 终端：开发效率提升利器

终端几剑客：iTerm2＋Zsh/Oh-My-Zsh+VIM＋Tmux。

没用 MacBook 以前在 Windows 上终端工具用的是 XShell，然后配合 Tmux，远程操作效率也还不错。

但是 iTerm2 使用起来真的我只能用一个字形容：爽。分屏太任性了，而且复制终端文本比 Tmux 方便。

```
    command + d : 水平分屏
    command + shift + d: 垂直分屏
    command + w : 关闭当前分屏
```

文本三巨头就不用多说了，和在 Linux 上操作是一致的。

其中，oh-my-zsh 安装配置：
```
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
    ＃或者
    sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
    ＃编辑 Zsh 配置文件
    vi ~/.zshrc
    # 选择随机主题
    ZSH_THEME="random" ＃ 主题目录在：~/.oh-my-zsh/themes/ 下
    source ~/.zshrc # 使配置文件生效
    # 查看系统的 shell
    cat /etc/shells
```

我认为 Zsh 的最大的方便之处在于简化了文件系统中的跳转，比如进入 WWW 可以不输入 cd WWW 而是直接输入 WWW，此外还使在终端下查看文件变得简单快速，比如只输入 l 就可以查看文件详情。

其他方便之处还在于对 git 仓库对智能识别，非常人性化。

### 免密码远程连接 Linux

不借助 SSH 工具直接在终端免密登录 Linux。

首先生成一个 SSH Key：

```
    cd ~ && mkdir .ssh && cd .ssh
    ssh-keygen -t rsa -C 'code0809@163.com' -f id_rsa_SoulMate94
```

注意：这里给 id_rsa_SoulMate94 的权限必须只能是某个特定用户可读可写，其他角色都不要给任何权限，比如要为 root 用户配置 ssh 远程登录则必须确保他的私钥文件权限是 chmod 0600 id_rsa_root，否则在执行 ssh -T git@git.HOSTNAME.ext 的时候警告你”私钥文件未受保护”（UNPROTECTED PRIVATE KEY FILE），从而导致 SSH 免密配置失败。

然后继续在本地用户的 .ssh 文件夹下新建配置文件：
```
    vim config
    # 输入类似如下配置
    Host github.com
    HostName github.com
    User SoulMate94
    Port 22
    IdentityFile ~/.ssh/id_rsa_SoulMate94
```

这里尤其需要注意的是用户名是否合法，比如检查：是否存在目标服务器上，名称是否含大小写等，用户名是否是 Git 注册邮箱。Host 和 HostName 也可以是 IP。

以后如果需要连接某台 Linux 服务器（包括 Git 服务器），只需要将生成色 id_rsa_cxl.pub 追加到服务器合法用户的 Home 目录下面的 .ssh 文件夹下的 authorized_keys 文件中即可，即要具备以下四个条件：
 - 服务器上要存在 SoulMate94 这个用户。
 - 在这个用户的 Home 目录下面的 .ssh 文件夹下的 authorized_keys 中含有该用户的 ssh 公钥。
 - 在想要连接到目标服务器的本地／远程用户的 Home 目录下的 .ssh 文件夹下面存在 config 文件并配置正确。
 - 主机地址正确解析，并且 config 文件中指定的私钥文件路径正确。

### 别名让终端操作更简单

比如上面已经配置好了免密码 ssh 远程服务器了，然后每次登录的时候虽然不再需要输入用户名和密码，但是仍然要输入类似 ssh root@remote_host.ext 这样长的命令，这时候为这条命令起个别名就显得非常简单快捷了。比如：
```
    # 本次终端会话期有效(暂时) 
    alias aly="ssh root@YOUR_REMOTE_LINUX_IP_ADDR"
    # 连接远程 Linux
    aly
    ＃使别名永久生效
    vim ~/.bash_profile
    vim ~/.zshrc
    # 添加上面的别名命令添加到 bash 和 zsh 配置文件的末尾即可
    # 使配置文件生效
    source ~/.bash_profile
    source ~/.zshrc
```

### rzsz - 古老而简单粗暴上下载

```
    brew install lrzsz
```

其中由于 rzsz 使用的协议在一些终端，比如在 iTerm 上并不原生支持，因此需要额外的配置，其中在 Tmux 上由于协议不同导致无法支持。

```
    # 下载 iTerm2 触发脚本
    git clone https://github.com/mmastrac/iterm2-zmodem.git
    # 将脚本 copy 到 /usr/local/bin 下面
    cp iterm2-zmodem/iterm2-recv-zmodem.sh /usr/local/bin/
    cp iterm2-zmodem/iterm2-send-zmodem.sh /usr/local/bin/
    # 赋予可执行权限
    chmod a+x /usr/local/bin/iterm2*.sh
```

**注意：** 假如 lrzsz 不是用 brew 安装，就必须修改脚本里面的 rz，sz 的安装路径了。

最后配置 iTerm2 里的 triggers：进入 iTerm => Preferences => Profiles => Default => Advanced => Triggers => Edit（也可使用快捷键 Command ＋,），然后新添两行内容如下：

```
    # 上传文件的触发规则
    Regular expression: rz waiting to receive.\*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-send-zmodem.sh
    Instant: checked
    # 下载文件的触发规则
    Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
    Instant: checked
```

注意脚本的路径一定要一致正确。然后只需记住 2 条命令即可：

```
    # 上传文件
    rz
    # 下载文件
    sz file1 file2 ...
```

若 iTerm2 更新到 3.0.0+则需要到 GitHub 下载最新的触发脚本重新配置。


### PHP+MySQL+Nginx

```
    # 服务先安装
    brew install mysql
    # MySQL 开机启动
    ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
    # 安装成功后开启 MySQL 安全机制
    /usr/local/opt/mysql/bin/mysql_secure_installation
    # 然后输入密码和一些选项, 启动 MySQL
    sudo mysqld
    # 查看一下MySQL运行情况
    ps aux | grep mysql
    ###### PHP 扩展先安装
    # 添加 PHP 扩展库
    brew update
    brew tap homebrew/dupes
    brew tap josegonzalez/homebrew-php
    # 选择 PHP 安装选项
    brew install php55 --with-fpm --with-gmp --with-imap --with-tidy --with-debug --with-mysql --with-libmysql
    # // 由于 PHP5.4+ 后带有 php-fpm 了因此这里启用就好了
    # // 其中 PHP 选项可以通过 brew options php55 来查看
    # 安装 PHP 扩展库
    brew install php55-apcu\
    php55-gearman\
    php55-geoip\
    php55-gmagick\
    php55-imagick\
    php55-intl\
    php55-mcrypt\
    php55-memcache\
    php55-memcached\
    php55-mongo\
    php55-opcache\
    php55-pdo-pgsql\
    php55-phalcon\
    php55-redis\
    php55-sphinx\
    php55-swoole\
    php55-uuid\
    php55-xdebug;
    # 安装 Nginx
    brew install nginx --with-http_geoip_module
    # // brew 安装 nginx 后的网站目录在：
    # //       /usr/local/Cellar/nginx/1.10.0/html
    # // 或    /usr/local/opt/nginx/html
    # 启动 nginx
    sudo nginx
    # Nginx 配置测试
    nginx -t
    # 重新加载配置|重启|停止|退出 nginx
    nginx -s reload|reopen|stop|quit
    # 也可以使用 Mac 的 launchctl 来启动|停止
    launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
    launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
    ＃ Nginx 开机启动
    ln -sfv /usr/local/opt/nginx/*.plist ~/Library/LaunchAgents
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
    # 监听 80 端口 (需要 root 权限)
    sudo chown root:wheel /usr/local/Cellar/nginx/1.10.0/bin/nginx
    sudo chmod u+s /usr/local/Cellar/nginx/1.10.0/bin/nginx
```

### 替换系统默认环境变量

```
    # 为了避免在使用 PHP 的时候执行系统默认的PHP版本需要替换环境变量
    echo 'export PATH="$(brew --prefix php55)/bin:$PATH"' >> ~/.bash_profile  #for php
    echo 'export PATH="$(brew --prefix php55)/sbin:$PATH"' >> ~/.bash_profile  #for php-fpm
    echo 'export PATH="/usr/local/bin:/usr/local/sbib:$PATH"' >> ~/.bash_profile #for other brew install soft
    source ~/.bash_profile
```

### 测试 brew 安装的 PHP 是否已优先启用

```
    # brew 安装的php 他在/usr/local/opt/php55/bin/php
    php -v    
    # Mac 自带的PHP
    /usr/bin/php -v   
    # brew 安装的php-fpm 他在/usr/local/opt/php55/sbin/php-fpm
    php-fpm -v
    # Mac 自带的php-fpm
    /usr/sbin/php-fpm -v
    # 开机启动 php-fpm
    ln -sfv /usr/local/opt/php55/*.plist ~/Library/LaunchAgents
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.php55.plist
```

### 测试 php-fpm 是否运行在通过 brew 安装的版本上

```
    # 测试 php-fpm 配置
    php-fpm -t
    # 启动 php-fpm
    php-fpm -D
    php-fpm -c /usr/local/etc/php/5.5/php.ini -y /usr/local/etc/php/5.5/php-fpm.conf -D
    # 关闭 php-fpm
    kill -INT `cat /usr/local/var/run/php-fpm.pid`
    # 重启php-fpm
    kill -USR2 `cat /usr/local/var/run/php-fpm.pid`
    # 也可以用 brew 命令来重启 php-fpm，不过官方已不推荐
    brew services restart php55
    # 还可以用这个命令来启动 php-fpm
    launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php55.plist
    # 启动 php-fpm 之后，确保它正常运行监听 9000 端口
    lsof -Pni4 | grep LISTEN | grep php
    # PHP-FPM 开机启动
    ln -sfv /usr/local/opt/php55/*.plist ~/Library/LaunchAgents
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.php55.plist
```

### 关联 Nginx 和 PHP

```
  vim /usr/local/etc/nginx/nginx.conf
  # ...
  location / {
  root   /Users/cxl/WWW;
  index index.php index.html index.htm;
  autoindex on;
  }
  # ...
  # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
  #
  location ~ \.php$ {
  root           html;
  fastcgi_pass   127.0.0.1:9000;
  fastcgi_index  index.php;
  fastcgi_param SCRIPT_FILENAME /Users/cxl/WWW/$fastcgi_script_name;
  include        /usr/local/etc/nginx/fastcgi_params;
  }
  # ...  
```

其中的 /Users/cxl/WWW/ 代表的是网站所在目录，建议都在用户家目录下操作，并更改目录所有者为当前用户，这样可以避免每次都要 sudo 。

然后修改 php-fpm 的配置文件：

```
    vim /usr/local/etc/php/5.5/php-fpm.conf
```

找到 pid ，去掉注释 pid = run/php-fpm.pid，此后 php-fpm 的 pid 文件就会自动产生在 /usr/local/var/run/php-fpm.pid，Nginx 的 pid 文件也在那。

### 别名
为了后面管理方便，将命令 alias 下，vim ~/.bash_aliases 输入以下内容：

```
    alias nginx.start='launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist'
    alias nginx.stop='launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist'
    alias nginx.restart='nginx.stop && nginx.start'
    alias php-fpm.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php55.plist"
    alias php-fpm.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.php55.plist"
    alias php-fpm.restart='php-fpm.stop && php-fpm.start'
    alias mysql.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist"
    alias mysql.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist"
    alias mysql.restart='mysql.stop && mysql.start'
    alias redis.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.redis.plist"
    alias redis.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.redis.plist"
    alias redis.restart='redis.stop && redis.start'
    alias memcached.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.memcached.plist"
    alias memcached.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.memcached.plist"
    alias memcached.restart='memcached.stop && memcached.start'
    # 让快捷命令生效
    echo "[[ -f ~/.bash_aliases ]] && . ~/.bash_aliases" >> ~/.bash_profile     
    source ~/.bash_profile
    # 创建站点目录到主目录，方便快捷访问
    ln -sfv /var/www ~/WWW
```

### 可能会遇到的问题

> 1.PHP 编译过程中如果遇到 configure: error: Cannot find OpenSSL's错误，执行xcode-select --install 重新安装一下 Xcode Command Line Tools。
> 2.如果 nginx 报 403 错误，请检查配置文件中的 root 指定的文件路径是否具有递归访问权限，如果终点路径可访问而父目录没有访问权限，那么同样会 403，赋予目录权限最好的方式是：更改所有者为将会访问目录的用户，然后设置为 0755。

## MPV - 命令行播放器

苹果自带的 QuickTime 播放器虽然很不错了，但是一些常见视频格式并不支持，比如 avi

mpv 是一款具有程序员个性的播放器。但是通过 brew 安装也很简单：

```
    brew tap mpv-player/mpv
    brew install --HEAD --with-bluray-support --with-libdvdread --with-little-cms2 --with-lua --with-bundle mpv
    ＃ 播放命令
    mpv /path/to/vedio.avi
    ＃ 将编译好的 mpv 链接到 Launchpad
    brew linkapps MPV
    ＃ 也可以自己配置 mpv
    vim ~/.config/mpv/mpv.conf
    # 比如调节音量加减的快捷键设置
    UP         add volume       1
    DOWN       add volume       -1
```

## 其他
```
    # 安装开发包
    brew install wget watch tmux cmake openssl imagemagick graphicsmagick gearman geoip readline autoconf multitail source-highlight autojump zsh-completions sshfs 
    # 升级系统自带 vim
    brew install ctags macvim --env-std --override-system-vim
    # brew cask install alfred appcleaner firefox google-chrome phpstorm sublime-text sequel-pro sketch mplayerx thunder qq
    # rar 格式的解压缩工具
    brew list | grep "rar"
    brew install unrar
    unrar x|e filename.rar # 主要用于解压从别处下载的文件, *x 系统建议使用 tar.gz
    # 7z 解压缩工具
    brew install p7zip
    p7zip x filename.7z
    # 科学上网利器 － shadowsocks
    <https://github.com/shadowsocks/shadowsocks>
```

总之，通过 Homebrew 安装的软件可以直接在终端通过命令来开启，比如通过 brew 安装的 sublime text 就可以直接使用 subl filename.ext 的方式在终端中快速打开，这对于我这种终端重度使用者来说很方便。

### 默认 Apache／PHP 配置
 - Apache 网站默认路径：/Library/WebServer/Documents
 - Apache 配置默认路径：/private/etc/apache2/httpd.conf
 - PHP 默认配置路径：/etc/php.ini.default
 
### Markdown 博客 － 不仅是一种知识管理方式

我有两份博客：GitHub 上的 Hexo 和 Coding.NET 上的 Hexo，每次在不同的地方写完东西后都要手动提交两份，很是累人，因此必须借助 Shell 命令别名来一步到位：

```
    vim ~/.zshrc
    alias post="bash ~/WWW/li/autocmt.sh && bash ~/WWW/SoulMate94.github.io/autocmt.sh"
    # 其中 ~/WWW/hexo/autocmt.sh 的内容是：
    cd ~/WWW/hexo
    git add -A
    git commit -m "backup hexo"
    git push origin master # 备份博客源码到 master 分支
    hexo clean
    hexo g
    hexo d # 发布文章到 coding-pages 分支
    
    
    # ~/WWW/SoulMate94.github.io/autocmt.sh" 的内容是：
    cd ~/WWW/SoulMate94.github.io
    git add -A
    git commit -m "Github post"
    git push origin master
```

于是我只需要输入 post 命令就可以提交两份博客了，虽然很简单，但是很实用。

注意上面必须先进入 hexo 博客所在目录，否则 hexo 命令执行会报错。

Markdown 工具也推荐 Typora，十分简洁，优雅，所见即所得却又并非传统双屏“书写＋预览”模式，多的不多说，体验一把就知道了。

## 附录：brew／cask 常用命令
```
    brew update                        # 更新 brew 可安装包，建议每次执行一下
    brew search php55                  # 搜索 php5.5
    brew tap josegonzalez/php          # 安装扩展 <gihhub_user/repo>   
    brew tap                           # 查看安装的扩展列表
    brew install php55                 # 安装 php5.5
    brew remove  php55                 # 卸载 php5.5
    brew upgrade php55                 # 升级 php5.5
    brew options php55                 # 查看 php5.5 安装选项
    brew info    php55                 # 查看 php5.5 相关信息
    brew home    php55                 # 访问 php5.5 官方网站
    brew services list                 # 查看系统通过 brew 安装的服务
    brew services cleanup              # 清除已卸载无用的启动配置文件
    brew services restart php55        # 重启 php-fpm
    brew cask search         # 列出所有可以被安装的软件
    brew cask search php     # 查找所有和 php 相关的应用
    brew cask list           # 列出所有通过 cask 安装的软件
    brew cask info phpstorm  # 查看 phpstorm 的信息
    brew cask uninstall mpv  # 卸载
```

## 参考
 - [Homebrew ( GitHub )](https://brew.sh/index_zh-cn.html)
 - [全新安装Mac OSX 开发者环境 同时使用homebrew搭建 PHP，Nginx ，MySQL，Redis，Memcache … … (LNMP开发环境)](https://segmentfault.com/a/1190000000606752)
 - [[Mac]安装和简单配置MPV播放器](https://imhy.zbyzbyzby.com/wordpress/?p=1167)
 - [如何重置 Mac 上的系统管理控制器 (SMC)](https://support.apple.com/zh-cn/HT201295)
 - [Reset the DNS cache in OS X](https://support.apple.com/en-us/HT202516)
 