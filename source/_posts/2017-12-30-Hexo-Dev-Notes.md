---
title: Hexo  开发记录
date: 2017-12-30 11:42:50
categories: Hexo
tags: [Hexo, 博客]
---

> 使用Hexo记录学习与生活中一些事情

# 准备工作

- 开发必备
  - 安装`Node.js`
  - 安装`Git`

<!--more-->
  
## 安装`Node.js`

- **Linux/Mac**
  - `cURL`:
  ```
    curl https://raw.github.com/creationix/nvm/master/install.sh | sh
  ```
  - `Wget`:
  ```
    wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
  ```
  - 安装完成后，重启终端并执行下列命令即可安装 `Node.js`。
  ```
   nvm install stable
  ```
- **Windows**
  - 直接下载
  
## 安装Git

- **Windows:** 下载并安装 `git`;

- **Mac:** 使用 `Homebrew`, MacPorts ：brew install git;或下载 安装程序 安装

- **Linux (Ubuntu, Debian):** `sudo apt-get install git-core`

- **Linux (Fedora, Red Hat, CentOS):** `sudo yum install git-core`


# 本地测试

```
    # 下载 Hexo
    npm install -g hexo
    npm insatll -g hexo-cli
    
    # 创建 Hexo 博客目录
    mkdir hexo
    
    # 进入 Hexo 博客目录并初始化 Hexo 博客站点
    cd hexo
    hexo init
    
    # 安装依赖
    npm install
    
    # 生成静态博客文件
    hexo generate
    
    # 运行 Hexo 服务
    hexo server
```


> 此时，在浏览器中输入 `localhost:4000` 就可以看到一个默认的 `Hexo` 博客了。


**如何启动多个博客?**

```
    hexo s -p 8888   // 端口号自己设置
```

# 部署到远程服务器

## 1、配置部署参数

在 `hexo/_config.yml` 中找到 `Deployment`，然后添加如下内容：

```
# Deployment 站点部署到github配置
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:SoulMate94/SoulMate94.github.io.git
  branch: master
```


其中，仓库地址需要灵活修改，仓库分支固定是 `gitcafe-pages`，可见 `GitCafe` 和 `GitHub` 也是一样都提供免费的 `Pages` 服务。

**说明**

这里为了防止后面在部署博客的时候重复输入用户名和密码，这里必须填写 `SSH` 协议而不是 `HTTPS` 的仓库地址，而且事先必须为部署的 `Git` 远程仓库服务配置好 `SSH` 密钥。

## 2、部署到 `Github`

首先，在 Github 注册账户并创建项目。需要注意的是，项目名和用户名要一致，并且要创建公开项目而不是私有项目。
然后运行：

```
    hexo generate
    
    # 安装 hexo-deployer-git
    npm install hexo-deployer-git --save
    
    # 开始部署到远程服务器
    hexo deploy
```

### 新建、更新、预览、同步你的博客

```
    hexo n 'name'
    hexo g
    hexo s
    hexo d
```


**说明**

每次修改本地文件后，都需要运行 hexo generate 才能使最新改动生效，每次执行命令时，都要在站点根目录下，即是站点配置文件 _config.yml 所在的目录 ( 不是主题配置文件 )。

# 发布文章

```
    git add -A    # 这里可根据自己情况添加文件
    git commit -m "adding"    # 这里的注释灵活填写
    git push alias master    # 这里的远程仓库别名和分支根据自己情况填写
    hexo clean    # 为了防止部署上次的博客内容建议先清除再重新生成
    hexo g
    hexo d
```

**简单操作: `hexo g && hexo d`**

# 配置 主题

首先当然也是先下载 `Hiker` 主题源码到 `Hexo` 博客的 `themes` 文件夹下面，即 `themes/next/`，`Hexo` 博客的所有主题都放到 `themes` 文件夹下。

```
    1.从GitHub上获取代码 
    git clone https://github.com/iTimeTraveler/hexo-theme-hiker.git themes/hiker
    
    2. 把Hexo主目录下的 _config.yml 文件中的字段 theme 修改为 hiker.
    # Extensions
    ## Plugins: https://hexo.io/plugins/
    ## Themes: https://hexo.io/themes/
    #theme: landscape
    theme: hexo-theme-hiker
    
    3. 更新
    cd themes/Hiker
    git pull
```

然后，对 `Hexo` 的配置主要就是配置根目录下的 `_config.yml` 文件，称作 **站点配置文件** 。

而对 `Hiker` 以及其他 `Hexo` 主题的配置主要就是配置主题所在文件夹下的 `_config.yml` 文件，称作 **主题配置文件**。

## 网站图标

在 `themes/next/source` 文件夹下放一个你事先准备好的 `favicon.ico `就行了。

## 侧边栏头像

在 `themes/next/source/images` 文件夹下放一个你事先准备好的 `default_avatar.jpg` 就行了。

此外，还可以放置 `avatar.jpg`，然后通过 `/uploads/avatar.jpg` 来引用。

或者在 `themes/next/source/images` 文件夹下新建一个 `uploads` 目录，里面存放 `avatar.jpg`，然后通过 `/uploads/avatar.jpg` 来引用。

编辑站点配置文件，新增字段 `avatar`，其值设置将成为头像的链接地址。

如果想要头像显示圆角效果，需要找到 `layout/_macro/sidebar.swig`，然后搜索 `site-author-image`，在这个 `<img>` 标签中加上 `style="border-radius:100%;"` 即可。


## 简体中文

`Hexo` 博客搭建好后默认是繁体，可以通过配置站点配置文件中的 `language` 为 `zh-CN` 即可。


## 侧边栏社交链接

编辑 站点配置文件，新增字段 `social`，然后添加社交站点名称与地址即可。例如：

```php
    # Social links
    social:
      GitHub: https://github.com/SoulMate94
      Google+: https://plus.google.com/110046839213865610992
```

摘要只需在 Hiker 主题配置文件中启用摘要和配置摘要的字数：

```php
# Automatically Excerpt
auto_excerpt:
  enable: true
  length: 150
```

此外，除了自动截取指定字数的文字作为文章摘要，还有种方式就是在写文章中使用 <!-- more --> 标签，代表的意思就是其前面的完整内容将作为摘要。

这样比自动截取摘要有个好处就是不会出现 ... 。

***注意***

如果采用的是自动截取摘要，则尽量不要使用代码段作为文章开头，否则博客首页将比较难看，手机端效果也很差。



# 扩展
- 全局搜索
  - npm install --save hexo-generator-search 
```
    _config.yml添加
    search: 
          path: search.xml
          field: post
```

**如果报错: --save 放置最后**

# FAQ

## 怎么创建标签云页面 ?

### 1、新建一个页面
```
# 命名为 tags
hexo new page "tags"
```

### 2、设置页面头信息
编辑刚新建的页面，将页面的类型设置为 tags ，主题将自动为这个页面显示标签云。页面内容如下：
```
title: TagCloud
date: 2016-01-24 11:44:35
type: "tags"
---
```

## 远程部署不成功 ?
- **Git**
  - `npm install hexo-deployer-git --save`
- **Heroku**
  - `npm install hexo-deployer-heroku --save`
- **Rsync**
  - `npm install hexo-deployer-rsync --save`
- **OpenShift**
  - `npm install hexo-deployer-openshift --save`
- **FTPSync**
  - `npm install hexo-deployer-ftpsync --save`

## 部署到Github 每次发布都出现404 ?

**在`source`文件夹下创建`CNAME`文件**

```
    存放域名: blog.caoxl.com
```

***REDEME.md同上***

## Hiker 主题下的 文章目录不能滑动

- 找到 `themes/hexo-theme-hiker/source/css/_partial/article.styl`

**或者 全局搜索`.toc-fixed`**

找到以下代码修改即可:

```
    .toc-fixed
      position: fixed;
      top: 30px;
      margin-top: 0;
      max-height: 97%;
      overflow: scroll;
      z-index: 1;
```

修改 `max-height` 和 `overflow` 即可

# 附录：Hexo 常用命令

```hexo
hexo chean    # 删除 public 目录
hexo g <=> hexo generate
hexo d <=> hexo deploy
hexo s <=> hexo server
hexo server -p 4444    # 更改 Hexo 服务器端口
hexo server -i 192.168.1.1    # 更改 Hexo 服务器 IP
hexo n <=> hexo new
```

# 参考

- [Hexo 文档](https://hexo.io/zh-cn/docs)
- [Hexo 常见问题](http://theme-next.iissnan.com/faqs.html#read-more)
- [Hexo 常见问题解决方案](http://wp.huangshiyang.com/hexo%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)
 
 
 