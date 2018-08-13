---
title: 修改Sublime Text 图标
date: 2018-02-27 14:59:33
categories: DevTool
tags: DevTool
---

看 `Sublime Text` 的应用程序图标不爽已经很久了，偶然看到一篇如何替换 App 图标的方法，便动手改了下，记录操作过程如下。


<!-- more -->

# 操作步骤

## 下载新图标

下载一个图标，[推荐 Dribbble](https://dribbble.com/shots/1827862-Yosemite-Sublime-Text-Icon)

最好就是 `.icns`的文件

## 打开图标文件夹

可以执行以下命令：

```
    open /Applications/Sublime\ Text.app/Contents/Resources/
```

## 替换图标

将下载的图标文件替换掉 `Sublime Text.icns` 文件


## 更新图标

此时，因为缓存的原因，替换重新打开 `Sublime Text App` 后可能未即时生效，可执行以下命令强制刷新图标：

```
touch /Applications/Sublime\ Text.app
touch /Applications/Sublime\ Text.app/Contents/Info.plist
```

重启即可看到新图标了。