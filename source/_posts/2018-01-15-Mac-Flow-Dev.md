---
title: Mac 流
date: 2018-01-15 17:26:21
categories: Mac
tags: Mac
---

## Mac 流
> 关于使用 Mac 过程中值得记录的都写在这里。

## 文件系统
 - **空格预览／cmd + Y：** 几乎可以预览一切文件
 - **文件关联：** 
   - 针对当前文件：** 右键 => 打开方式
   - 针对所有相同文件后缀的文件：右键 => 显示简介 => 打开方式 => 全部修改
 - **上下左右：** 当前路径中按顺序切换文件
 - **command+上／下：** 返回上层目录／进入当前选中目录
 - **command+i：** 显示文件信息
 - **command + m + i：** 同时显示多个文件信息
 - **command+k：** 连接到服务器
 - **command+delete：** 移动到废纸篓
 - **command+shift+delete：** 清空废纸篓
 - **alt+command+delete：** 彻底删除
 - **文件／文件夹显示模式：** command + 1~4
 - **文件／文件夹排序模式：** command + ctrl + 0~7
 

<!--more-->
 
 
## 全局／相似
 - **command + ,：** 设置
 - **command+w：** 关闭窗口
 - **command+q：** 退出程序
 - **command+n：** 新建当前窗口副本
 - **command+ m：** 最小化当前窗口（按住 command 点 tab 选中目标窗口后按住 alt，然后松开 command 恢复 ）
 - **command+h：** 隐藏当前窗口（用 command + tab 恢复 ）
 - **command + `：** 在程序内未隐藏的子窗口中切换
 - **command + alt + h:** 隐藏当前窗口外的所有窗口
 - **command + alt + m h:** 隐藏当前所有窗口
 - **ctrl + command + d:** 查看鼠标选中／光标选中的英文单词含义；或着预览超链接
 - **F11／fn+F11：** 显示桌面
 - **ctrl + left/right：** 切换桌面
 - **tab + space：** 选择切换 + 确认
 - **三指点按：** 查询单词与数据检测
 
 
## 字体
```
    brew tap caskroom/fonts
    brew cask install font-fira-code
    brew cask install font-roboto-mono
    brew cask install font-mononoki
    brew cask install font-inconsolata
    brew cask install font-consolas-for-powerline
    brew cask install font-dejavu-sans-mono-for-powerline
```


## Homebrew
macOS 的软件包及其依赖管理工具。

```
    # 安装 Homebrew
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

**常用命令：**
 - 查看已安装软件包
 ```
    brew list
    brew cask list
 ```
 
 - 搜索软件包
 ```
    brew search node
    # brew search --help
 ```
 
 - 查看软件包安装选项
 ```
    brew options node
 ```
 
 - 安装
 ```
    brew install node    # 从预编译二进制包中安装
    brew install --build-from-source node    # 从源码编译安装
    brew reinstall licecap      # 重装已删除的应用程序
    brew cask install chrome    # 下载带 GUI 的软件包
 ```
 
 > 如果有个软件包名是在 cask 仓库下，比如：caskroom/cask/mysql-shell，则既可以通过 brew install caskroom/cask/mysql-shell 安装也可以通过 brew cask install mysql-shell 安装。
 
 - 完全卸载
 ```
    brew uninstall node
 ```
 
 - 只从环境变量中卸载
 ```
    brew unlink node
 ```
 
 - 恢复到环境变量
 ```
    brew link node
 ```
 
 - 查看当前 homebrew 概况
 ```
    brew info
 ```
 
 - 查看软件包属性
 ```
    brew info node
    brew cask info font-roboto
    # 格式化输出
    brew install jq    # commandline JSON processor
    brew info --json=v1 node | jq
    brew info --json=v1 node | jq "map(select(.linked_keg != null) | .name)"
 ```
 
 - 查看／切换软件仓库
 ```
    brew tap    # 查看当前 homebrew 使用到的软件仓库 (homebrew/core 为其主仓库)
    brew tap homebrew/nginx    # 切换到某个功能仓库
    # 一些有趣的仓库：https://docs.brew.sh/Interesting-Taps-and-Forks.html
 ```
 
 - 数据统计
 ```
    brew analytics        # 查看 homebrew 数据统计状态
    brew analytics off    # 关闭 homebrew 数据统计
    brew analytics regenerate-uuid    # 重新生成 UUID
    # 可在这里查看：https://brew.sh/analytics/
 ```
 
## 开发
 - **Dash:** API 即时搜索体验
 
     到<u> https://kapeli.com/dash </u>下载安装即可。
     
     安装结束可以根据需要下载 docset 和 cheatsheet，开发相关的，几乎都有。
     
     在使用 Alfred/Spotlight 搜索 dash 时可以使用伪协议格式进行搜索关键词或快捷键，比如：dash://php:
 
 - **Docker:** 引领容器时代。
    <u>https://docs.docker.com/docker-for-mac/。</u>
 
 - **Vagrant：** 所有环境不一致“流浪汉们”的归属
   - 制作和线上服务器相同的测试／开发环境，在团队间打包分发
   - 快速创建／销毁同等环境，不再担心环境不小心被破坏
   
 - **Sublime Text：** 最爱的 GUI 编辑器。
  see: [Sublime Sublime Text.](http://blog.cjli.info/2016/10/12/Sublime-Sublime-Text/)
  
 - **PhpStorm：** 可能是目前最专业最酷的 PHP IDE
   - 配置远程编辑+自动同步代码
   配合 Vagrant Workflow 可以避免本地安装多余的开发环境。
   
   最终想要实现的效果是：本地只编码／版本控制，Vagrant 管理的环境中测试代码，尽可能减少团队成员间环境不同带来的冲突，和本地与生产环境不同带来的问题等。
   
   PhpStorm 开启远程编辑，流程根据版本不同，大致如下：
  ```
    # 1. 添加一个远程服务器路径 通常是测试环境全局虚拟主机所在的根路径 也可以配置为某个虚拟主机所在的跟路径
    File => Default Settings => Bsuild,Execution,Deployment => Deployment => `+`
    Name => centos.dev    # 远程服务器的名字
    # 2. 配置好「连接选项」、「控制编码格式」 和 「根 URL」
    ## 配置后可以点击【Test SFTP connection】测试配置是否有误
    ######  选择 Connection 子标签
    Type => SFTP    # 假设 Vagrant 测试环境为 Linux
    SFTP host => 虚拟机的 IP    # 要求主机已安装好 host-only 网卡驱动
    Port => 22
    Root Path => /data/wwwroot/    # 测试环境中代码的根路径
    User name => root
    Auth Type => Password    # 这里也可以使用 OpenSSH 但是在可以记住密码的情况下 最终也不用每次都手动输入密码
    Password  => ******      # 勾选【Save password】
    Advanced options => Control encoding => UTF-8
    # 可选
    ## 如果填了 这里的根 URL 就是测试环境中所配虚拟主机的域名
    ## 如果留空 则表示当前配置为多个虚拟主机公共的跟路径 可以在 Mappings 里为每个虚拟主机配置一个 Web URL
    Web server root URL => http://sample.dev
    # 连接测试通过后 点击 【OK】保存此配置
    # 3. 配置本地项目和测试环境网站根目录的路径映射关系
    ######  选择 Mappings 子标签
    Local path => /www/sample.dev    # 要同步的本地路径
    Deployment path on server 'centos.dev' => sample.dev    # 要同步的远程路径
    Web path on server 'centos.dev' => /sample.dev    # 远程服务器上的该同步项目的可访问的 Web URL
    # 如果还有该远程服务器还有其他项目 则可以点击【Add another mapping】继续新增 Mapping
    # ...
  ```
 配置自动同步代码：Tools => Depolyment => 勾选上 Automatic upload (always)。
  
  手动同步：alt + command + y。
  
  *说明:*
  这里虽然可以通过配置共享文件来实现 Host 和 Client 系统的代码同步，但是实际操作中经常出现共享文件夹速度很慢，Windows(smb) 和 macOS（nfs）的配置文件不能共享等的现象。因此，为了节省开发时间，个人的最佳实践还是配置远程编辑。
  
- **Navicat：** 强大的关系型数据库全能工具
  - 将模型导出为 SQL
  
  点击「文件」=> 「导出 SQL」=> 「导出到文件」=> 「选择文件路径+填写导出的文件名+选择导出的选项」=> 「确定」。
  
  需要注意的一点是，在「导出 SQL」时，不能直接只输入一个文件名就完事了，而是要点击右侧的 ... 按钮，然后选择路径和填写文件名才可以导出成功。否则会 Navicat 报错（11.2.10）。
  

## 录入

### Typora：简单高效 Markdown
 - 文档大纲：shift + command + b
 - 源码模式和预览模式：command + /
 - Focus 模式：shift + command + r
 - 图片：alt/option + command +i
 - 高亮：shift+ command + h <=> ==TEXT==
 - 插入表格：command+t
 - 查找：command+f
 - 查找并替换：command+alt+f
 - 粘贴纯文本：shift + command + v
 
### 自带输入法：原配最好
 - 中文输入法下调用符号：shift +alt/option + b
 - 输入符号：alt+键盘符号／字母
 - 显示字符键盘：ctrl + command + space （可更改）
 - 打开关闭手写输入：ctrl + shift + space
 - 打开 emoji 表情：cmd + ctrl + space
 
### 一些设置
 - 开启 Capslock 切换输入法
 作为国人，中英切换的频率是很频繁的，而常用的快捷键 CMD／Space／Ctrl 等又给了 Spotlight 和 Alfred 等工具，后来在设置中偶然找到了可以使用 Capslock 键切换中英输入，试了几下居然操作挺 6。于是在这里安利出来了：
 
 系统设置 => 键盘 => 输入法 => 勾选下方「使用大写锁定键切换“美国”输入模式（长按以启用全大写键入）」。
 
 - 取消自动纠正拼写和大写字词的首字母
 系统设置 => 键盘 => 文本 => 取消右侧「自动纠正拼写」和「自动大写字词的首字母」。（个人觉得很烦而已 ):）
 
 - 取消连按两下空格键插入句号
 系统设置 => 键盘 => 文本 => 取消右侧「连按两下空格键插入句号」。
 
 
## 终端
### 移除 “Last login” 提示信息
```
    cd ~
    touch .hushlogin
```

## 快捷键
### Terminal
 - Command + K：清屏（真正的清屏）
 - Command + T：新建标签
 - Command +W：关闭当前标签页
 - Command + S：保存终端输出
 - Command + D：水平分隔当前标签页
 - Command + Shift + D：取消水平分隔
 - Command + shift + {／}：向左/向右切换标签
 - Control + U／K：删除光标前/后 所有单词
 - Control + Y：撤销上个操作
 
### iTerm2
 - 新建标签：command+t
 - 新建分屏：command+d（竖）；command + shift + d（横）
 - 标签间切换：command+左／右
 - 分屏间切换：command + [/]
 - 全屏切换：command + enter
 - 清除当前行：ctrl + u
 - 前进后退（左右）：ctrl + f/b
 - 跳至行首／行尾：fn+左／右；ctrl + a／ctrl+e
 - 向光标左／右方向跳一个单词：esc + b／f
 - 清屏：ctrl + l ；command + r（只是把光标重置到终端左上角）
 - 剪切板历史：command+shift+ h
 - 从光标处删至命令行尾：ctrl + k
 - 从光标处删至命令行首：ctrl + h
 - 粘贴至光标后：ctrl + y
 - 搜索命令历史：ctrl + r
 - 复制
   - 选中即复制
   - 搜索后使用 shift 往右选择或 shift + tab 往左选择
 - 光标聚光灯：command + /
 
### zsh & oh-my-zsh

```
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

 - plugins=(git, thefuck, zsh-autosuggestions)
 - alias
 ```
    alias cls="clear"
    alias ip="echo 'LAN :' `ipconfig getifaddr en0` && echo 'WAN :' `dig +short myip.opendns.com @resolver1.opendns.com`"
    alias chmd="chown -R www:www ./"
    alias szsh="source ~/.zshrc"
    alias idu="du -h -d 1 ."
    alias cpssh="pbcopy < ~/.ssh/id_rsa.pub"
    alias rmf="rm -rf $1"
    alias rmals="sudo rm /private/var/log/asl/*.asl"
    alias lline='fc -ln -1 | awk '\''{$1=$1}1'\'' | pbcopy'
    # Dev
    alias gs="git status | lolcat"
    alias ga="git add $1"
    alias gad="git add -A"
    alias gb="git branch"
    alias gbd="git branch -D $1"
    alias gbb="git checkout -b $1"
    alias gbbb="git checkout -b $1 $2"
    alias gct='read comments && git commit -m "$comments"'
    alias gcta="git commit --amend"
    alias gplm="git pull origin master"
    alias gplb="git pull origin $1"
    alias gfb="git fetch origin $1"
    alias gfm="git fetch origin master"
    alias gfa="git fetch origin"
    alias gpsm="git push origin master"
    alias gpsb="git push origin $1"
    alias gpsd="git push origin --delete $1"
    alias gcm="git checkout master"
    alias gcb="git checkout $1"
    alias gmm="git merge master"
    alias gmb="git merge $1"
    alias gdev='git commit -a --amend -m "deving at `date`"'
    alias dg='git log --pretty=format:"- %s" --author=cxl --since=9am > ~/.today.md && subl ~/.today.md'
    vagrant_home="/dl/vagrant/"
    alias vup="cd $vagrant_home && vagrant up"
    alias vssh="ssh root@centos.dev"
    alias vh="vagrant halt"
    alias vre="vagrant reload"
    alias phpa="php -a"
    alias phps7="php -S 0.0.0.0:7777"
    alias phps8="php -S 0.0.0.0:8888"
    alias phps9="php -S 0.0.0.0:9999"
    alias phpat="php artisan tinker"
    hexo_path="/dl/self/SoulMate94.github.io"
    alias phexo="sh $hexo_path/auto.sh $hexo_path"
    alias pswd="/dl/self/tellmepasswd/TellMePasswd.php"
    alias ss="subl ~/.ss-in-bulk/ss.cnf"
    alias sspac="subl ~/.ShadowsocksX/gfwlist.js"
    alias ssupac="subl ~/.ShadowsocksX/user-rule.txt"
    alias vps="ssh user@host"
 ```
 
### VIM

#### awesome vimrc
懒人专用。自动配置 vim。

```
    git clone https://github.com/amix/vimrc.git ~/.vim_runtime
    sh ~/.vim_runtime/install_awesome_vimrc.sh
    # update
    cd ~/.vim_runtime
    git pull --rebase    
```

#### plugins

如果不用别人的，就需要手动安装需要的插件。

vim 插件一般是 .vba 或者 .vim 文件，一般拷贝到 ~/.vim/plugins 下面就行了，具体参考 http://www.vim.org/ 的插件安装说明。

个人认为常用且有趣的插件有：
    - Devlife
    - GuidedTour
    - NERD_tree
    - tortoiseTyping
    - SrcExpl
    - indentLine
    - markdown
    - taglist
    - matrix
    
### 命令行工具
#### 命令帮助：man 和 cheat
 ```
    man htop
    brew install cheat
    cheat -v
    cheat tar
 ```

#### 解压缩
 - rar
 ```
    brew install unrar
    # 解压单压缩文件
    unrar x file.rar
    # 解压分卷压缩文件
    unrar  x -o- -y file.part1.rar </path/to/save>
 ```
 
 - 7z
 ```
    brew install p7zip
    7z e file.7z
 ```

#### 统计
 - cloc：Count Lines of Code
 ```
    cloc /path/to/proj
 ```
 
 - gitstats
 ```
    brew install gnuplot
    brew install libjpeg
    https://github.com/hoxu/gitstats && cd gitstats
    /.gitstats /path/to/proj /path/to/output
 ```
 
 - fswatch
   - 后台同步
 ```
    /usr/bin/nohup fswatch -o /env/vagrant/www/eme.dev/ | xargs -n1 /env/vagrant/www/eme.dev/tools/deploy/rsync-start > /dev/null 2>&1 &
    # rsync-start
    /usr/local/bin/rsync -avz --progress --no-o --no-g -m --chmod=Du=rwx,Dgo=rx,Fu=rw,Fog=r -F --exclude='.git/' /env/vagrant/www/eme.dev/ root@eme.dev:/data/wwwroot/eme.dev
 ```
 
 #### 其他
 ```
    brew install lolcat cmatrix cowsay
 ```
 
### 常用指令
 - 打开指定路径的 Finder：open /path/to/open
 - 锁屏：
 ```
    /System/Library/CoreServices/Menu\ Extras/User.menu/Contents/Resources/CGSession -suspend
 ```
 
## 生产力
### Alfred

带有「workflow」的超级 Spotlight 搜索，各种 workflow 需要自行安装仔细体会。个人使用的工作流有：
 - Kill Progress：kill progress_name
 - DevDocs
 - Dynamic File Search
 - Encode/Decode
 ```
    encode string
    decode string
 ```
 - Colors：#
 - GitHub
 - Dict
 ```
    cc {word} @ {dict}    # 注意如果修改 hot key 后 @ 实效 建议恢复默认热键
 ```
 - StackOverflow
 - Sublime Text
 - Douban
 ```
    book BOOK_NAME
    movie MOVIE_NAME
    music MUSIC_NAME
 ```
 - Timezones：tz
 - Domainr
 - Terminal Finder
 - VirtualBox
 - IP Address：ip
 - Emoji：emoji
 
 若搜索 alfred 则打开 Alfred 偏好设置。
 
 ### Moom
 键盘党最爱的窗口管理之一。
 
 ### Cheatsheet
 快捷键作弊条。长按 command 键呼出快捷键列表。
 ```
    brew install caskroom/cask/cheatshee
 ```
 <u>https://www.mediaatelier.com/CheatSheet/。</u>
 
 ### Spectacle
 开源的窗口管理工具，简单实用。我只在默认设置的基础上改了 6个快捷键：
  - Center：shift + command + c
  - Fullscreen: ctrl + shift + f
  - Make Larger: command + =
  - Make Smaller: command+ -
  - Left Screen：shift + ctrl + Left
  - Right Screen：shift + ctrl + Right
  
 其他保持默认，个人使用过程中基本上没有冲突。其他快捷键如下：
 - ctrl + command + left/right：以全屏1／4 窗口紧贴左／右上角
 - ctrl + shift + command + left/right：以全屏1／4 窗口紧贴左／右下角
 - alt + command + left/right/up/down：以全屏1／2 窗口紧贴左／右／上／下
 
 ### Harmmerspoon
 源码级自定义全局快捷键。使用 lua 写配置，举例如下：
 ```
    # init.lua
    hs.hotkey.bind({"cmd", "alt", "ctrl"}, "H", function()
      hs.notify.new({title="Hammerspoon", informativeText="Hammerspoon is working."}):send()
    end)
    hs.hotkey.bind({"cmd", "alt", "ctrl"}, "Left", function()
      local win = hs.window.focusedWindow()
      local f = win:frame()
      f.x = f.x - 100
      win:setFrame(f)
    end)
    hs.hotkey.bind({"cmd", "alt", "ctrl"}, "Right", function()
      local win = hs.window.focusedWindow()
      local f = win:frame()
      f.x = f.x + 100
      win:setFrame(f)
    end)
    hs.hotkey.bind({"cmd", "alt", "ctrl"}, "Up", function()
      local win = hs.window.focusedWindow()
      local f = win:frame()
      f.y = f.y - 100
      win:setFrame(f)
    end)
    hs.hotkey.bind({"cmd", "alt", "ctrl"}, "Down", function()
      local win = hs.window.focusedWindow()
      local f = win:frame()
      f.y = f.y + 100
      win:setFrame(f)
    end)
 ```
 
 这样，按住 ctrl + alt + command + 上／下／左／右，当前窗口就可以进行 四个方向上每 100 像素／次的微调了。
 
 ### InsomniaX
 可以在不接交流电源（AC power）的情况下开启合盖模式。
 
 ### ShortCat
 
 系统级的 Vimium
 
 Chrome 有款插件叫 Vimium，作用是通过快捷键来跳转网页也链接。
 
 ShortCat 的作用就像 Mac 系统内全局的 Vimium，可以通过快捷键来快速定义屏幕范围内指定的按钮。
 
 
 ### Automator
 Mac 自带的工作流定义工具。
 
 这玩意一直没怎么研究过，偶然间在网上看到有人介绍用这个完成一些批量命名的操作，顿时来了兴趣。
 
 #### 批量命名 Finder 文件
  - 打开 Automator
  - 文稿类型选择「文件夹操作」
  - 找到「给 Finder 项目重新命名」工作流
  - 添加到右侧工作区 (是否在副本上操作自行决定)
  - 然后定义工作流（批量操作的格式）的细节
  - 命名 && 保存
  
 然后在 Finder 里面选择多个文件或者文件夹 => 右键选择文件夹操作设置 => 找到刚刚定义的工作流 => 输入一些参数即可让此工作流开始工作。
 
 此后，只要在 Finder 中选中多个文件夹，右键属性中都会出现 “给X个项目重新命名” 的选项了，确实非常方便
 
 #### 给 Finder 增加「在终端中打开」
  - 打开 Automator
  - 选择「服务」
  - 右侧顶部的「“服务”收到选定的」 选择「文件或文件夹」，位于选择「Finder.app」
  - 左侧顶部操作下找到「文件和文件夹」，拖到右边工作区，其右侧子项目选中「打开 Finder 项目」
  - 右侧中部「打开 Finder 项目」下面的「打开方式」选择终端／iTerm 2.app
  - 命名 && 保存
  
## 系统设置

### 重命名账户名称
不得不说，Apple 设计的很多功能是一个需要你自己去体会的，我常常用着用着突然发现，原来这个功能是有的。

系统偏好设置 => 用户与群组 => 点按锁按钮以进行更改 => 鼠标移动到用户列表中的某个用户 => 右键高级 => 此页即可修改用户信息。

### MacBook Pro 遇到过的问题

#### 别人遇到的问题:
 - 插入 SD 卡后重启变卡，进入登录界面时键盘触摸屏不能使用
 - 五国死机过
 - 自动重启
 - 键盘灯总是常亮，每次登录时亮度调整已经是 0，但是只有先增加再减少后才能生效
 - 键盘一直开启大写设置无效
 - 声音时大时小
 
#### 自己遇到的问题:
 暂无...
 
 

## 参考

 - [Alfred 3 Workflows](https://github.com/zenorocha/alfred-workflows)
 - [Alfred 2 Workflows List](http://www.alfredworkflow.com/)
 - [程序员如何优雅地使用macOS？](https://www.zhihu.com/question/20873070)
 - [重置 Mac 上的系统管理控制器 (SMC) - Apple 支持](https://support.apple.com/zh-cn/HT201295)
 - [Mac OS X 配置指南 | Mac OS X Setup Guide](https://mac-setup.wildflame.org/)