---
title: Windows 常用命令记录
date: 2018-08-08 11:47:52
categories: Caoxl
tags: [Windows,命令]
---

这篇文章将记录比较实用的一些 Windows 命令，将不间断更新。

<!-- more -->


# 唤起命令行

```
    windows键 + F
    然后输入 cmd
    输出help 可以查看命令列表
```

# 命令行操作

```
    cls               // 清屏
    winver            // 查看windows版本
    write             // 写字板
    mspaint           // 画图板
    magnify           // 放大镜
    mmc               // 打开控制中心
    regedit           // 打开注册表
    calc              // 启动计算器
    osk               // 打开屏幕键盘
    userinit          // 打开我的文档
    
    esc               // 清除当前命令行
    F7                // 显示命令历史记录，以图形列表窗的形式给出所有曾经输入的命令，并可用上下箭头键选择再次执行该命令
    alt + F7          // 清除所有层级输入的命令历史记录
    F8                // 搜索命令的历史记录，循环显示所有曾经输入的命令，直到按下回车键为止
    F9                // 按编号选择命令，以图形对话框方式要求您输入命令所对应的编号(从0开始)，并将该命令显示在屏幕上
    
    ctrl + h          // 删除光标左边的一个字符
    ctrl + c / ctrl + break // 强行终止命令执行
    ctrl + m          // 表示回车确认键
    alt + printscreen // 截取屏幕上当前命令窗里的内容
```

# 电源管理

```
    shutdown -s -t 0  // 立即关机
    shutdown -a       // 取消关机
    rononce -p        // 15秒关机
```

# 文件管理

```
    type             // 查看文本文件内容
    type NUL > [path/to/file] // 在某个路径下新建文件
    fc               // 查看两个文本文档的不同之处
    cd               // 跳转到下一个目录
    copy             // 复制
    move             // 移动
    del              // 删除文件/文件夹
    dir              // 显示文件
    mkdir            // 新建文件夹
    rmdir            // 删除文件夹下所有文件及文件夹
    icacls * /reset  // 清除(还原)当前目录下的所有文件/文件夹的权限设置
    ie4uinit -show   // 清除系统图标缓存
    services.msc     // 系统服务管理
    attrib           // 设置文件属性 [详见这篇文章](https://paugram.com/tech/usb-disk-custom-icon.html#%E8%AE%BE%E7%BD%AE%E6%96%87%E4%BB%B6%E5%B1%9E%E6%80%A7)）
```

# 网络维护

```
    ping (IP地址/域名)     // 向指定服务器进行通信, 测试能否连接
    ipconfig /flushdns   // 清除 DNS 缓存
    ssh (用户名)@(IP地址)  // 建立 SSH 连接, 需要在控制面板添加功能
```

# 系统激活

```
    slmgr.vbs -xpr        // 查询系统是否批量激活还是永久激活
```

# 硬件维护

- 调取电池使用记录

```
    // 仅用于笔记本
    powercfg /BATTERYREPORT
    C:\Users\用户名\battery-report.html
```

输入后会自动保存到用户文件夹内，直接键入文件名打开。

其中 `DESIGN CAPACITY` 是设计容量， `FULL CHARGE CAPACITY` 是近期充满电池的容量。
想要知道电池还有多少毫安，通过公式 mwh（毫瓦时） / 电池电压（V） = 毫安（mAh） 即可得到


# 参考

- [windows常用命令行命令](https://blog.csdn.net/liubinblog/article/details/9060259)
- [Windows批处理(cmd/bat)使用小记](https://www.zybuluo.com/yangfch3/note/338252)