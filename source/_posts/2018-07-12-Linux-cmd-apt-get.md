---
title: Linux 命令 「apt-get」
date: 2018-07-12 16:06:42
categories: Linux cmd
tags: [Linux]
---

> `apt-get`命令是`Debian Linux`发行版中的APT软件包管理工具, 类似于`CentOS`下的`yum`.

<!-- more -->

# 语法

```
    apt-get (选项) (参数)
```

# 选项

- `-c`:   指定配置文件

# 参数

- 管理指令: 对`APT`软件包的管理操作
- 软件包:  指定要操纵的软件包

# 实例

- 安装一个新软件包

```
    apt-get install package_name
```

- 卸载一个已安装的软件包(保留配置文件)

```
    apt-get remove package_name
```

- 卸载一个已安装的软件包(删除配置文件)

```
    apt-get -purage remove package_name
```

- 删除已经删掉的软件

```
    apt-get autoclean apt
```

- 删除已安装的软件的备份

```
    apt-get clean
```

- 更新所有已安装的软件包

```
    apt-get upgrade
```

- 将系统升级到新版本

```
    apt-get dist-upgrade
```