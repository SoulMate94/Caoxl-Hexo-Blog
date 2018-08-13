---
title: Linux 命令 「ifconfig/ifup/ifdown」
date: 2018-08-03 09:49:41
categories: Linux cmd
tags: [Linux]
---

> **ifconfig命令**被用于配置和显示Linux内核中网络接口的网络参数

<!-- more -->

# `ifconfig`

## 语法

```
    ifconfig (参数)
```

## 常见参数

- `add`<地址>: 设置网络设备IPv6的IP地址
- `del`<地址>: 删除网络设备IPv6的IP地址
- `down`: 关闭指定的网络设备
- `io_addr`<I/O地址>: 设置网络设备的I/O地址
- `up`: 启动指定的网络设备
- `IP`地址: 指定网络设备的IP地址
- 网络设备: 指定网络设备的名称

## 实例

### 显示网络设备信息(激活状态的):

```
    [root@caoxl ~]# ifconfig
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 172.31.45.94  netmask 255.255.240.0  broadcast 172.31.47.255
            ether 00:16:3e:01:a4:4f  txqueuelen 1000  (Ethernet)
            RX packets 4367  bytes 1362027 (1.2 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 4090  bytes 1498312 (1.4 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 36  bytes 4896 (4.7 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 36  bytes 4896 (4.7 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

**说明:**

- **eth0**表示第一块网卡，其中`ether`表示网卡的物理地址，可以看到目前这个网卡的**物理地址(MAC地址）** 是`00:16:3e:01:a4:4f`

- **inet**用来表示网卡的**IP地址**，此网卡的**IP地址**是`172.31.45.94`，**广播地址broadcast**:`172.31.47.255`，**掩码地址netmask**:`255.255.240.0`。

- **lo**是表示主机的**回环地址**，这个一般是用来测试一个网络程序，但又不想让局域网或外网的用户能够查看，只能在此台主机上运行和查看所用的网络接口。比如把 `httpd`服务器的指定到回环地址，在浏览器输入`127.0.0.1`就能看到你所架WEB网站了。但只是您能看得到，局域网的其它主机或用户无从知道。
  - 第一行: `UP`（代表网卡开启状态）`RUNNING`（代表网卡的网线被接上）`MULTICAST`（支持组播）`MTU:1500`（最大传输单元）：1500字节
  - 第二行: 网卡的IP地址、掩码地址
  - 第三行:
  - 第四、五行: 接收数据包情况统计
  - 第六、七行: 发送数据包情况统计
  
### 启动关闭指定网卡:

```
    ifconfig eth0 up     // 启动网卡eth0
    ifconfig eth0 down   // 关闭网卡eth0
```

**ssh登陆linux服务器操作要小心，关闭了就不能开启了，除非你有多网卡。**

### 为网卡配置和删除`IPv6`地址:

```
    ifconfig eth0 add 33ffe:3240:800:1005::2/64    # 为网卡eth0配置IPv6地址
    ifconfig eth0 del 33ffe:3240:800:1005::2/64    # 为网卡eth0配置IPv6地址
```

### 用`ifconfig`修改`MAC`地址:

```
    ifconfig eth0 hw ether 00:AA:BB:CC:dd:EE
```

### 配置`IP`地址:

```
    ifconfig eth0 192.168.2.10
    ifconfig eth0 192.168.2.10 netmask 255.255.255.0
    ifconfig eth0 192.168.2.10 netmask 255.255.255.0 broadcast 192.168.2.255
```

### 启用和关闭`arp`协议:

```
    ifconfig eth0 arp    # 开启网卡eth0的arp协议
    ifconfig eth0 -arp   # 关闭网卡eth0的arp协议
```

### 设置最大传输单元:

```
    ifcongig eth mtu 1500  # 设置能通过的最大数据包大小为 1500 bytes
```

# `ifup`

> **ifup命令**用于激活指定的网络接口。

## 语法

```
    ifup (参数)
```

## 参数

- 网络接口: 要激活的网络接口

## 实例

```
    ifup eth0   # 激活eth0
```

# `ifdown`

> **ifdown命令**用于禁用指定的网络接口。

## 语法

``` 
    ifdown (参数)
```

## 参数

- 网络接口: 要禁用的网络接口

## 实例

```
    ifdown eth0   # 禁用eth0
```