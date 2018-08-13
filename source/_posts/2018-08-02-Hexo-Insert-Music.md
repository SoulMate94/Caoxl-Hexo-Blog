---
title: Hexo 插入音乐
date: 2018-08-02 14:48:10
categories: Hexo
tags: [Hexo]
---

> 正如你在归档和文章正文看到的, 我引入了一个音乐播放器

<!-- more -->

# 装插件

 首先我们需要通过命令行安装以下两款插件（当然装一个就可以了，如果一个实现不了再装另一个），这两款插件我们都可以从hexo的官方网站上查询到：[Hexo 插件](https://hexo.io/plugins/)

```
    npm install hexo-tag-dplayer 
    npm install hexo-tag-aplayer
```

# 添加音乐

在我的博客中我选择添加的是网易云音乐。首先我们打开网易云音乐的主页，然后搜索你喜欢的音乐，然后进入音乐的详情页面，点击 **生成外链播放器**

- 复制`iframe`插件`HTML`代码

```
    <div style="position:absolute; top:10%; left:1%; width:76%;">
      <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=260 height=52 src="//music.163.com/outchain/player?type=2&id=25706279&auto=1&height=32"></iframe>
    </div>
```

**以上代码复制到主题文件夹下, 想放在哪个区域，就把代码复制到实现那块区域的模板文件里**


为了使得页面更加美观，我给这块代码自定义加了 div，这样方便给播放器设定样式，只要稍微懂一点前端知识的筒子，都可以随心所欲的加 `css`样式。这里需要提一下，播放器设定好后，默认是：打开页面即播放音乐；如果不喜欢默认打开音乐的筒子，也可以通过把代码中的 `auto=1` 修改为 `auto=0` 来关闭它。


 这样就`Done~~~`