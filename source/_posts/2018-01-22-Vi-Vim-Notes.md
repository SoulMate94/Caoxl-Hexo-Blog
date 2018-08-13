---
title: Vi / Vim 基础知识
date: 2018-01-22 10:46:47
categories: DevTool
tags: [Vi, Vim, Linux]
---

# Vi 基础知识

- 三种模式：命令模式、输入模式、末行模式。

<!--more-->

## 命令模式

vi 启动后默认进入的是命令模式。输入 i、a 都能进入编辑模式。无论在任何模式下要按 esc 都进入命令模式。

**常用命令 :**

- `x:`删除当前光标所在的字符
- `nx:`删除当前行包含光标后n个字符
- `D:`删除当前行光标后的所有的字符
- `dd:`删除当前光标所在的行
- `ndd:`删除当前行(包括当前行)后面的n行
- `yy:`复制当前行
- `p:`粘贴
- `u:`撤销

## 末行模式

命令模式下输入 `:` 即可进入末行模式。末行模式下可以执行的操作有：

**常用命令 :**

`:set nonu` 取消行号
`:n` 将光标定位到第n行
`:$` 回到文件的最末行
`:/string` 把string字符串进行高亮显示
`:nohls`  取消高亮显示
`:w` 保存
`:q` 退出
`:wq` 保存并退出或输入 :x也行
`:wq!`	强制保存并退出
`:q!` 强制退出
`:[range]s/源字符串/目标字符串/[pattern]` 替换字符串
 
## vi或者vim的常用指令

- `vi  filename ：` 编辑一个文件（如果文件不存在则自动创建，否则打开）
- `vi  filename1  filename2：`编辑多个文件
  - `:next`  :n编辑下一个文件
  - `:prev` 编辑上一个文件
- `vi  +n 	filename：`编辑一个文件，光标位于第n行
  - `+1：`第一行
  - `+$：`最后一行 

# 配置 Vi

配置 Vi 就是对 ~/.vimrc 文件的配置，如下是我现在正常使用的配置内容：

```shell
    # 在启动 vim 时 当前用户家目录下的 .vimrc 文件会被自动读取
    # 所以一般情况下把 .vimrc 文件创建在当前用户的根目录下
    # 该文件可以包含一些设置甚至脚本
    
    # 去掉有关 vi 一致性模式 避免以前版本的一些 bug 和局限
    set nocompatible
    
    # 显示行号
    # set nummber
    
    # 检测文件的类型
    filetype on 
    
    # 记录历史的行数
    set history=1000 
    
    # 背景使用黑色
    set background=dark 
    
    # 语法高亮显示
    syntax on
    
    # 下面两行在进行编写代码时 在格式对齐上很有用
    # 自动对齐 也就是把当前行的对齐格式应用到下一行
    set autoindent
    # 依据上面的对齐格式 智能的选择对齐方式
    set smartindent
    
    # 第一行设置 tab 键为 4 个空格
    # 第二行设置当行之间交错时使用 4 个空格
    set tabstop=4
    set shiftwidth=4
    
    # 去除 vim 的 GUI 版本中的 toolbar
    set guioptions-=T
    
    # 设置匹配模式 类似当输入一个左括号时会匹配相应的那个右括号
    set showmatch
    
    # 在编辑过程中，在右下角显示光标位置的状态行
    set ruler
    # 关闭搜索匹配时的高亮显示
    # set nohls
    
    # 搜索提示
    set incsearch
    
    # 设置所在行提示
    set cul
    
    # 设置文件编码
    set fileencoding=utf-8
```

# Vi 快捷键

**说明：快捷键在非编辑模式下有用。**

vi 编辑器中ctrl+s 为终止屏幕输出，即是停止回显，ctrl+q 恢复屏幕输出。

- `dd` 剪切某行内容到 VIM 缓冲区

- `yy` 复制某行内容到 VIM 缓冲区

- `gg` 跳至文件头

- `GG` 跳至文件尾

- `P` 向光标上粘贴 VIM 缓冲区中的 **最新** 内容

- `p` 向光标下粘贴 VIM 缓冲区中的 **最新** 内容

- `ZZ` 保存并退出

- `dd` 剪切本行



# 参考

- <u>[Vim Cheat Sheet](https://vim.rtorr.com/)</u>