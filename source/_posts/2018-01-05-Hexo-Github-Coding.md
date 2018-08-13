---
title: Hexo 在Github/Coding中搭建
date: 2018-01-05 11:44:10
categories: Hexo
tags: [Hexo, 博客]
---

# 在Github/Coding上部署Hexo
> 将hexo部署到代码库的同时指向自定义域名

## 博客部署
- Github: http://blog.caoxl.com
- Coding: http://life.caoxl.com


<!--more-->


## 部署必备
- 服务器 (推荐阿里云)
- 域名
- Github账号
- Coding(码市)账号

## 部署过程
- 在服务器上添加域名解析
  - Github
  ```
    1. 添加一个CNAME记录类型 记录值为:SoulMate94.github.io
    2. 添加两个A记录类型(@) 记录值为:GithubIP地址:
    192.30.252.153  192.30.252.154
  ```
  - Coding
  ```
    1. 添加一个CNAME记录类型 记录值为:soulmate94.coding.me
  ```

- 在Github/Coding上打开域名指向
  - Github
  
  ```
    在 项目SoulMate94.github.io中/Settings/GitHub Pages 添加一个Custom domain (如blog.caoxl.com)
  ```
  
  - Coding
  
  ```
    在 我的项目/代码/Pages服务 中添加自定义域名 (如life.caoxl.com)
  ```

## FAQ
- 问题: hexo部署后，CNAME会被自动删除，怎么办？
- 答: 将需要上传至github的内容放在source文件夹，例如CNAME、favicon.ico、images等。

- 问题: 怎么部署REDEME.md到github上又消失了
- 答: 在source文件夹创建后 ,在_config,yml中添加skip_render: README.md 即可 

 
## 参考
- Hexo开发详细另见:[Hexo-Dec-Record](http://blog.caoxl.com/2017/12/29/Hexo-Dec-Record/)
