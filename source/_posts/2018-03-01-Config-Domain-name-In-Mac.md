---
title: Mac配置IP域名映射
date: 2018-03-01 10:23:28
categories: Mac
tags: Mac
---

使用Mac开发需要知道的一些小知识。


<!-- more -->

# 本地开发

hosts文件修改： 
    1. 在应用程序里面打开终端(terminal) 
    2. 输入 sudo vi /etc/hosts 
    3. 接着输入 i 进入编辑模式 
    4. 将添加的域名,ip拷贝进去，我随便取了个域名：127.0.0.1    test.dev 
    5. 编辑完成之后,按esc,输入 ": wq" 

可能有些朋友碰到hosts为只读，不能修改，解决方法如下： 
    1. 打开finder, 快捷键：shift+command+g 前往文件夹 “/etc” 
    2. 找到hosts文件托到桌面修改，再把/etc下源文件删除，把桌面修改好的拖进/etc。 

最后在终端：ping test.dev ,如果能ping通到127.0.0.1，说明映射成功 

