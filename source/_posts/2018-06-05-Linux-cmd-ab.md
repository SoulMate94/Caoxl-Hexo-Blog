---
title: Linux 命令 「ab」
date: 2018-06-06 14:08:40
categories: Linux cmd
tags: [Linux,ab]
---

# `ab`

**ab命令**是Apache的Web服务器的性能测试工具，它可以测试安装Web服务器每秒种处理的HTTP请求。

<!-- more -->

## 安装

- `CentOS`

```linux
    yum -y install httpd-tools
```

- `Ubuntu`

```linux
    apt-get install apache2-utils
```

## 查看版本/命令

- `ab -v` 查看版本
- `ab -help` 查看命令

# 命令参数

- `-A`: 指定连接服务器的基本的认证凭据
- `-c`: 指定一次向服务器发出请求数
- `-C`: 添加cookie
- `-g`: 将测试结果输出为:`gnuolot`文件
- `-h`: 显示帮助信息
- `-H`: 为请求追加一个额外的头
- `-i`: 使用`head`请求方式
- `-k`: 激活HTTP中的`keepAlive`特性
- `-n`: 指定测试会话使用的请求数
- `-p`: 指定包含数据的文件
- `-q`: 不显示进度百分比
- `-T`: 使用POST数据时,设置内容类型头
- `-v`: 设置详细模式等级
- `-w`: 以HTML表格方式打印结果
- `-x`: 以表格方式输出时,设置表格的属性
- `-X`: 使用指定的代理服务器发送请求
- `-y`: 以表格方式输出时,设置表格属性


# 使用

```linux
    ab -n 1000 -c 10 http://www.caoxl.com
```

> -n访问1000次, -c并发10个

**ab压力测试返回报文内容详解:**

```
    Server Software:        nginx               #服务器软件
    Server Hostname:        www.caoxl.com       #域名
    Server Port:            80                  #请求端口号
    
    Document Path:          /                   #文件路径
    Document Length:        3648 bytes          #页面字节数
    
    Concurrency Level:      10                  #请求的并发数
    Time taken for tests:   30.317 seconds      #总访问时间
    Complete requests:      1000                #请求成功数量
    Failed requests:        0                   #请求失败数量
    Write errors:           0
    Total transferred:      3898000 bytes       #请求总数据大小(包括header头信息)
    HTML transferred:       3648000 bytes       #html页面实际总字节数
    Requests per second:    32.98 [#/sec] (mean)#每秒多少请求,(服务器的吞吐量)
    Time per request:       303.173 [ms] (mean) #用户平均请求等待时间
    Time per request:       30.317 [ms] (mean, across all concurrent requests) # 服务器平均处理时间，也就是服务器吞吐量的倒数
    Transfer rate:          125.56 [Kbytes/sec] received #每秒获取的数据长度
    
    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:       31   38  55.2     35    1058
    Processing:    32  258 379.0    145    3986
    Waiting:       31  157 297.3     37    3730
    Total:         63  296 384.2    179    4021
    
    Percentage of the requests served within a certain time (ms)
      50%    179        #50%用户请求在179ms内返回
      66%    334        #66%用户请求在334ms内返回
      75%    435        #75%用户请求在435ms内返回
      80%    447        #80%用户请求在447ms内返回
      90%    678        #90%用户请求在678ms内返回
      95%    929        #95%用户请求在929ms内返回
      98%   1176        #98%用户请求在1176ms内返回
      99%   1860        #99%用户请求在1860ms内返回
     100%   4021 (longest request)
```