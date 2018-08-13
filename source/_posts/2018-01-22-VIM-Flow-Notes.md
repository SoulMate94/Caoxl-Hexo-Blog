---
title: VIM 流
date: 2018-01-22 10:42:16
categories: DevTool
tags: VIM
---

> 没有用过VIM的程序员不是好程序员

<!--more-->

# Buffer／缓冲区

- 查看缓冲区：`:ls` 或者 `:buffers`
> %a 代表当前缓冲区；# 代表上一个缓冲区

- 打开某个文件到缓冲区：`:e[dit] <path_to_file>`

- 安全退出缓冲区（需要先保存缓冲区内容）：`:bd[elete] <id>`

- 强制性退出缓冲区：`:bd[elete]! <id>`

- 缓冲区跳转：

  - 快捷键：`ctrl + ^`
  - 命令
    - 跳转到指定编号的缓冲区：`b <id, filename, expression>`
    - 上一个缓冲区：`bn[ext]`
    - 下一个缓冲区：`bp[revious]`
    - 跳转到文件名最匹配给定字符串的缓冲区（模糊匹配并打开）：`:b task`

# Window／屏幕／窗口

## 分屏／退出

- 水平分屏：`:sp[lit]`
  - 水平分屏并打开多个文件：`vim -O path/to/file1 path/to/file2 ...`
  - 水平分屏并以只读方式打开：`sv[iew] path/to/file`

- 新建窗口/垂直分屏：`:new` 或 `:sb` 或 `vs[plit]`
  - 垂直分屏并打开指定编号的缓冲区：`:sb 1` 或 `:vertical sb 1`
  - 垂直分屏并打开多个文件：`vim -o path/to/file1 path/to/file2 ...`
  - 右下垂直分屏并打开指定文件：`:vertical rightbelow sfind path/to/file`

- 安全退出窗口：`:q[uit]` 或者 `:clo[se]`


## 快捷键

```
    # 打开关闭
    Ctrl+w s        水平分割当前窗口
    Ctrl+w v        垂直分割当前窗口
    Ctrl+w q        关闭当前窗口
    Ctrl+w n        打开一个新窗口（空文件）
    Ctrl+w o        关闭出当前窗口之外的所有窗口
    Ctrl+w T        当前窗口移动到新标签页
    
    
    # 切换窗口 
    Ctrl+w h        切换到左边窗口
    Ctrl+w j        切换到下边窗口
    Ctrl+w k        切换到上边窗口
    Ctrl+w l        切换到右边窗口
    Ctrl+w w        遍历切换窗口
    Ctrl+w t        最上方的窗口
    Ctrl+w b        最下方的窗口
    
    
    # 移动窗口（注意大写）
    Ctrl+w H        向左移动当前窗口
    Ctrl+w J        向下移动当前窗口
    Ctrl+w K        向上移动当前窗口
    Ctrl+w L        向右移动当前窗口
    
    
    # 调整大小
    Ctrl+w +        增加窗口高度
    Ctrl+w -        减小窗口高度
    Ctrl+w =        统一窗口高度
```

# Tab／标签

## 打开关闭

- **VIM 外：** `vim -p path/to/file1 path/to/file2 ...`

- **VIM 内：** 

```
    :tabe[dit] {file}   edit specified file in a new tab
    :tabf[ind] {file}   open a new tab with filename given, searching the 'path' to find it
    :tabc[lose]         close current tab
    :tabc[lose] {i}     close i-th tab
    :tabo[nly]          close all other tabs (show only the current tab)s
```

## 移动

```
    :tabs         list all tabs including their displayed window
    :tabm 0       move current tab to first
    :tabm         move current tab to last
    :tabm {i}     move current tab to position i+1
```

## 跳转

- 命令模式下：

```
    :tabn         go to next tab
    :tabp         go to previous tab
    :tabfirst     go to first tab
    :tablast      go to last tab
```

- 正常模式下使用快捷键：

```
    gt            go to next tab
    gT            go to previous tab
    {i}gt         go to tab in position i
```

# 其他

- 命令帮助：`:h <command>`

- 文件跳转
  - 设置工作路径为当前 `VIM` 打开路径：`:set path=$PWD/**`
  - 文件名大小写区分查找并跳转：`:sfind filename`
  
- 调用外部命令：`:call system('date')`

- 重载文件
  - 放弃当前修改，强制重新载入：`:e!`
  - 重新载入所有打开的文件：`:bufdo e` 或 `:bufdo :e!`
  > `:bufdo` 命令表示把后面的命令应用到所有 `buffer` 中的文件

# 附录

![VIM 命令速查表](https://images0.cnblogs.com/blog/651196/201504/132336240577495.png)

# 参考

- <u>[Vim 多文件编辑：缓冲区](http://harttle.land/2015/11/17/vim-buffer.html)</u>
- <u>[如何用Vim搭建IDE？](http://harttle.land/2015/11/04/vim-ide.html)</u>
- <u>[Vim Cheat Sheet](https://vim.rtorr.com/)</u>
