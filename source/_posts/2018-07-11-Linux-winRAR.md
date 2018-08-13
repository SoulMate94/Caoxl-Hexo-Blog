---
title: Linux 压缩/解压缩
date: 2018-07-11 09:25:57
categories: Linux
tags: [Linux]
---

> Linux下的压缩解压缩命令集合

<!-- more -->

# `.tar`

> `tar`是打包, 不是压缩!

## 打包

```
    tar -cvf filename.tar DirName
```

## 解包

```
    tar -xvf filename.tar
```

# `.tar.gz` 和 `.gz`

## 压缩

```
    tar -zcvf filename.tar.gz DirName
```

## 解压

```
    tar -zxvf filename.tar.gz
```

# `.gz`

## 压缩

```
    gzip filename
```

## 解压

```
    gunzip filename.gz
    
    或
    gzip -d filename.gz
```

# `.zip`

## 压缩

```
    zip filename.zip DirName
```

## 解压

```
    unzip filename.zip
```

# `.rar`

## 压缩

```
    rar a filename.rar DirName
```

## 解压

```
    rar x filename.rar
```

# 常见问题

## 使用`rar`遇到问题

> **报错: error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file**

出现这种情况的原因是，`rar`命令需要`libstdc++`的`32`位版本，系统应该只安装`64`位版本，可以通过rpm查看是否

```
    [root@izj6c6djex81rijczh0t8yz winrar]# rpm -ql libstdc++ | cat -n
         1	/usr/lib64/libstdc++.so.6
         2	/usr/lib64/libstdc++.so.6.0.19
         3	/usr/share/gcc-4.8.2
         4	/usr/share/gcc-4.8.2/python
         5	/usr/share/gcc-4.8.2/python/libstdcxx
         6	/usr/share/gcc-4.8.2/python/libstdcxx/__init__.py
         7	/usr/share/gcc-4.8.2/python/libstdcxx/__init__.pyc
         8	/usr/share/gcc-4.8.2/python/libstdcxx/__init__.pyo
         9	/usr/share/gcc-4.8.2/python/libstdcxx/v6
        10	/usr/share/gcc-4.8.2/python/libstdcxx/v6/__init__.py
        11	/usr/share/gcc-4.8.2/python/libstdcxx/v6/__init__.pyc
        12	/usr/share/gcc-4.8.2/python/libstdcxx/v6/__init__.pyo
        13	/usr/share/gcc-4.8.2/python/libstdcxx/v6/printers.py
        14	/usr/share/gcc-4.8.2/python/libstdcxx/v6/printers.pyc
        15	/usr/share/gcc-4.8.2/python/libstdcxx/v6/printers.pyo
        16	/usr/share/gcc-4.8.5
        17	/usr/share/gdb
        18	/usr/share/gdb/auto-load
        19	/usr/share/gdb/auto-load/usr
        20	/usr/share/gdb/auto-load/usr/lib64
        21	/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.py
        22	/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyc
        23	/usr/share/gdb/auto-load/usr/lib64/libstdc++.so.6.0.19-gdb.pyo
```

说明没有安装`32`位的`libstdc++`版本，需要安全`32`位的，可以通过`yum`安装，先执行

```
    [root@izj6c6djex81rijczh0t8yz winrar]# yum list | grep libstdc++
    libstdc++.x86_64                          4.8.5-28.el7_5.1             @updates 
    libstdc++-devel.x86_64                    4.8.5-28.el7_5.1             @updates 
    compat-libstdc++-33.i686                  3.2.3-72.el7                 base     
    compat-libstdc++-33.x86_64                3.2.3-72.el7                 base     
    libstdc++.i686                            4.8.5-28.el7_5.1             updates  
    libstdc++-devel.i686                      4.8.5-28.el7_5.1             updates  
    libstdc++-docs.x86_64                     4.8.5-28.el7_5.1             updates  
    libstdc++-static.i686                     4.8.5-28.el7_5.1             updates  
   
   
    libstdc++-static.x86_64                   4.8.5-28.el7_5.1             updates  
```

发现`libstdc++.i686`(注有的可能是`i386`,`i586`), 然后执行安装命令

```
    [root@izj6c6djex81rijczh0t8yz winrar]# yum -y install libstdc++.i686
    
    Running transaction
      Installing : libgcc-4.8.5-28.el7_5.1.i686                                                   1/2 
      Installing : libstdc++-4.8.5-28.el7_5.1.i686                                                2/2 
      Verifying  : libgcc-4.8.5-28.el7_5.1.i686                                                   1/2 
      Verifying  : libstdc++-4.8.5-28.el7_5.1.i686                                                2/2 
    
    Installed:
      libstdc++.i686 0:4.8.5-28.el7_5.1                                                               
    
    Dependency Installed:
      libgcc.i686 0:4.8.5-28.el7_5.1                                                                  
    
    Complete!
```


