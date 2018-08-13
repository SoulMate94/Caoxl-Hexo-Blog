---
title: CentOS After Minimal
date: 2018-01-24 17:17:07
categories: Linux
tags: [Linux, CentOS]
---

这里对 CentOS 作为服务器个人需要做的事务列举出来。

**说明：** 由于 CentOS 多做服务器，服务器一般很少为了支持新硬件去更新内核，再加上现如今常用的阿里云之类的云服务器不能更新内核，所以这里暂时忽略内核的升级过程。

<!--more-->

# resolv.conf & hosts

```
    # resolv.conf
    vi /etc/resolv.conf
      ; Google
      nameserver 8.8.8.8
      nameserver 8.8.4.4
      ; DNSPod
      nameserver 119.29.29.29
      nameserver 182.254.116.116
      ; OpenDNS
      nameserver 208.67.222.222
      nameserver 208.67.220.220
      ; V2EX
      nameserver 178.79.131.110
      
      ; 114
      114.114.114.114
    service network restart
    ping github.com
    ping baidu.com
    # hosts (Just in case)
    vi /etc/hosts
      #207.97.227.239 github.com
      204.232.175.78 documentcloud.github.com
      204.232.175.94 gist.github.com
      107.21.116.220 help.github.com
      207.97.227.252 nodeload.github.com
      199.27.76.130  raw.github.com
      107.22.3.110   status.github.com
      204.232.175.78 training.github.co
      207.97.227.243 www.github.com
```

# 开启 EPEL/IUS/Remi Repos

```
    # !!! ensure root permission first
    # 1. EPEL
    yum install epel-release
    # Or
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-[5|6|7].noarch.rpm
    rpm -Uvh epel-release-*.rpm
    
    # 2. IUS
    wget https://centos[5|6|7].iuscommunity.org/ius-release.rpm
    rpm -Uvh ius-release*.rpm
    # Upgrade installed packages to IUS versions
    yum install -y yum-plugin-replace
    # eg: yum replace php --replace-with php53
    
    # 3. Remi
    wget http://rpms.famillecollet.com/enterprise/remi-release-[5|6|7].rpm
    rpm -Uvh remi-release-*.rpm
    # Enable Remi
    vim /etc/yum.repos.d/remi.repo    # enabled=1
    # Or
    yum --enablerepo=remi install php-tcpdf
    
    # update and check
    yum -y update
    yum groupinstall -y 'development tools'
    yum repolist
    
    # fastestmirror
    yum install fastestmirror
```

# 编译安装最新版 Git

> CentOS 6.8 使用 yum 安装 git 总是 1.7，要使用最新 Git 必须编译安装。

```
    # 1. Install Required Packages
    yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
    
    # 2. Download and Install Git
    cd /usr/src
    wget https://www.kernel.org/pub/software/scm/git/git-2.10.0.tar.gz
    tar xzf git-2.10.0.tar.gz
    cd git-2.10.0
    make prefix=/usr/local/git all
    make prefix=/usr/local/git install
    
    # 3. Setup Environment
    export PATH=$PATH:/usr/local/git/bin
    source /etc/bashrc
    # Or zsh etc.
    ln -s /usr/local/git/bin/git /usr/local/bin
    
    # 4. Check
    git --version
```

# Vim
    
```
    yum remove vi
    yum install -y vim
    git clone https://github.com/amix/vimrc.git ~/.vim_runtime
    sh ~/.vim_runtime/install_awesome_vimrc.sh
    # Include your staff like this
    git clone git@github.com:uguu-org/vim-matrix-screensaver.git ~/.vim_runtime/sources_non_forked/vim-matrix-screensaver
```

# zsh & oh-my-zsh && zsh-users

```
    yum install -y zsh
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
    # Or
    # sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
    
    cd ~/.oh-my-zsh/plugins
    git clone git://github.com/zsh-users/zsh-autosuggestions
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
    
    vim ~/.zshrc
    plugins=(git, zsh-autosuggestions, zsh-syntax-highlighting)
    source ~/.zshrc
```

# LEMPA

使用 [oneinstack](https://oneinstack.com/download/) 快速搭建 LEMPA 环境即可。

选择当前最新组合：php7.1.^ + mysql^5.7. + nginx^1.11. + Apache^2. + phpMyAdmin^4.6.*。

**选择合适的版本即可**

其中，如果想自定义每个项目的版本，可以在 `oneinstack/versions.txt` 里面指定你想要通过 `oneinstack` 安装的软件版本。

## phpMyAdmin 高级功能设置方法

1.首次登录 phpMyAdmin 后导入 `sql/create_tables.sq`l，即创建 `phpmyadmin` 这个数据库

2.取消 `config.inc.php` 中关于 `phpmyadmin` 的注释，搜索 `/* Storage database and tables */`，其下十几张表全部打开

# Python

由于依赖这货的东西太多了，为了减少麻烦，请保留 centos6.8 上原有的 python2.6 版本，要使用高版本的 python 请命名为 python2（表示最新 python2. 版本，python3（表示最新 python 3. 版本），然后 link 到 $PATH。

```
    # 更新 & 依赖安装
    # yum groupinstall -y 'development tools'
    yum install -y zlib-devel bzip2-devel openssl-devel xz-libs wget
    
    # 下载 & 解压
    wget http://www.python.org/ftp/python/2.7.9/Python-2.7.9.tar.xz
    xz -d Python-2.7.9.tar.xz
    tar -xvf Python-2.7.9.tar
    
    # 编译 & 安装
    cd Python-2.7.8
    ./configure --prefix=/usr/local
    make && make altinstall
    
    # PATH
    export PATH="/usr/local/bin:$PATH"
    # or 
    ln -s /usr/local/bin/python2.7  /usr/bin/python
    
    # setuptools
    wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-1.4.2.tar.gz
    tar -xvf setuptools-1.4.2.tar.gz
    cd setuptools-1.4.2
    python2.7 setup.py install
    
    # pip
    curl  https://bootstrap.pypa.io/get-pip.py | python2.7 -
    
    # 修复 yum [如果非要用 `python` 覆盖 2.6 版本的话]
    vim /usr/bin/yum
    #!/usr/bin/python => #!/usr/bin/python2.6
```

# NodeJS

```
    wget https://nodejs.org/dist/v4.5.0/node-v4.5.0-linux-x64.tar.xz
    xz -d node-v4.5.0-linux-x64.tar.xz
    tar -xf node-v4.5.0-linux-x64.tar /usr/local/src/node4.5
    ln -s /usr/local/src/node4.5/bin/node /usr/local/bin/
    rm -rf node-v4.5.0-linux-x64.tar.xz
```


# Ruby

```
    wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
    tar -zxvf ruby-2.3.1.tar.gz
    cd ruby-2.3.1
    # default installed to /usr/local, use `./configure --prefix=DIR` to change
    ./configure
    make && make install
```

# 其他安装清单

```
    yum install tmux lsof htop
```

# alias

```
    alias vi="vim"
    alias cls="clear"
    alias clshis="echo ''>~/.zshrc_history && echo '' > ~/.bash_history"
    alias chmd="chown -R www:www /data/wwwroot/"
    alias vzshrc="vim ~/.zshrc"
    alias szsh="source ~/.zshrc"
    alias sbash="source ~/.bash_profile"
    alias gad="git add . -A"
    alias gcmt='read comments && git commit -m "$comments"'
    alias gcm="git checkout master"
    alias gpullm="git pull origin master"
    alias vssh="vagrant ssh"
    alias vre="vagrant reload"
    alias vup="vagrant up"
    alias vgs="vagrant global-status"
    alias vhalt="vagrant halt"
```

# $PATH

```
    export PATH="/usr/local/bin:/usr/local/sbin:$PATH"
    export PATH="/root/.composer/vendor/bin:$PATH"
```

# FAQ

- **Could not retrieve mirrorlist**

> Your dns isn’t working. Check /etc/resolv.conf for valid and accessible nameserver lines. Use 8.8.8.8 if needs be.

- **Can’t locate ExtUtils/MakeMaker.pm” while compile git**

```
    yum install perl-devel
    # if the above not work, try also install the CPAN
    # yum install perl-CPAN
```

- **error: There was a problem with the editor ‘vi’**

```
    # Usually
    git config --global core.editor /usr/bin/vim
    
    # Or
    git config --global core.editor $(which vim)
```

- **登录 phpMyAdmin 后提示：The secret passphrase in configuration (blowfish_secret) is too short.**
    
```
    vim /path/to/phpMyAdmin/config.inc.php
    
    $cfg['blowfish_secret'] = '32位字符串'; # 默认不是 32 位长度 改为 32 位长度即可
```

- **yum Error:rpmdb open failed**

```
    mkdir /root/backups.rpm.mm_dd_yyyy/
    cp -avr /var/lib/rpm/ /root/backups.rpm.mm_dd_yyyy/
    
    rm -f /var/lib/rpm/__db*
    db_verify /var/lib/rpm/Packages
    rpm --rebuilddb
    yum clean all
    
    yum update
```

# 参考

- [Install EPEL, IUS, and Remi repositories on CentOS and Red Hat](https://support.rackspace.com/how-to/install-epel-and-additional-repositories-on-centos-and-red-hat/)

- [How to Install Git 2.8.1 on CentOS/RHEL 7/6/5 & Fedora](https://tecadmin.net/install-git-on-centos-fedora/#)

- [https://github.com/robbyrussell/oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

- [zsh-users](https://github.com/zsh-users)

- [Python包管理工具setuptools详解](http://yansu.org/2013/06/07/learn-python-setuptools-in-detail.html)

- [yum Error:rpmdb open failed](https://unix.stackexchange.com/questions/198703/yum-errorrpmdb-open-failed)