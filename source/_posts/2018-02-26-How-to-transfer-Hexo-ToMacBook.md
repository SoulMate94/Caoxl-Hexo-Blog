---
title: Hexo 更换电脑后怎么继续使用博客
date: 2018-02-26 10:30:55
categories: Hexo
tags: [Hexo]
---

近期需要将Windows上的东西转移到Mac。 需要在mac上继续使用Hexo博客

<!-- more -->

- 1. 将你原来电脑上已经配置好并生成的`hexo`目录拷到你的新电脑上，注意无需拷全部，只拷如下几个目录：

```
	 _config.yml
	 package.json
	 scaffolds/
	 source/
	 themes/
```

将这些目录放到一个目录下，如：hexo／


- 2. 在你的新电脑上首先配置hexo环境: [安装Node.js](https://nodejs.org/zh-cn/)

- 3. 安装hexo，执行命令：

```
	npm install -g hexo

	// 仅到当前目录
	npm install hexo
```

- 4. 安装好之后，进入hexo／目录

- 5. 模块安装，执行命令:

```
    npm install
    npm install hexo-deployer-git --save
    npm install hexo-generator-feed --save
    npm install hexo-generator-sitemap --save
```

- 6. 部署，执行命令:


```
	hexo g && hexo d
```