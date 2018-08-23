---
title: Linux 命令 「dmidecode」
date: 2018-08-17 09:11:46
categories: Linux cmd
tags: [Linux, dmidecode]
---

> **dmidecode命令**可以让你在Linux系统下**获取有关硬件方面的信息**。`dmidecode`的作用是将DMI数据库中的信息解码，以可读的文本方式显示。由于DMI信息可以人为修改，因此里面的信息不一定是系统准确的信息。`dmidecode`遵循`SMBIOS/DMI`标准，其输出的信息包括BIOS、系统、主板、处理器、内存、缓存等等。


<!-- more -->


# 语法

```
    dmidecode [选项]
```

# 选项

- `-d`: (default:/dev/mem)从设备文件读取信息, 输出内容与不加参数的标准输出相同
- `-h`: 显示帮助信息
- `-s`: 只显示指定DMI字符串的信息 (string)
- `-t`: 只显示指定条目的信息 (type)
- `-u`: 显示未解码的原始条目内容
- `--dump-bin file`: 将DMI数据转储到一个二进制文件中
- `--from-dump FILE`: 从一个二进制文件读取DMI数据
- `-V`: 显示版本信息


# `dmidecode`参数`string`及`type`列表：

## （1）Valid string keywords are：

- `bios-vendor`  (BIOS供应商)
- `bios-version`  (BIOS版本)
- `bios-release-date`  (BIOS发行时间)
- `system-manufacturer`  (系统制造商)
- `system-product-name`  (系统产品名称)
- `system-version`  (系统版本)
- `system-serial-number`  (系统序列号)
- `system-uuid`  (系统UUID)
- `baseboard-manufacturer`  (主板motherboard制造商)
- `baseboard-product-name`  (主板motherboard产品名称)
- `baseboard-version`  (主板motherboard版本)
- `baseboard-serial-number`  (主板motherboard序列号)
- `baseboard-asset-tag`  (主板motherboard资产标签)
- `chassis-manufacturer`  (机箱制造商)
- `chassis-type`  (机箱类型)
- `chassis-version`  (机箱版本)
- `chassis-serial-number`  (机箱序列号)
- `chassis-asset-tag`  (机箱资产标签)
- `processor-family`  (处理器家族)
- `processor-manufacturer`  (处理器的制造商)
- `processor-version`  (处理器版本)
- `processor-frequency`  (处理器的频率)

## （2）Valid type keywords are：

- `bios`  (BIOS)
- `system`  (系统)
- `baseboard`  (主板)
- `chassis`  (机箱)
- `processor`  (处理器)
- `memory`  (内存)
- `Cache`  (缓存)
- `connector`  (连接器)
- `slot`  (插槽)

## （3）type全部编码列表：

- `BIOS`  (BIOS)
- `System`  (系统)
- `Base Board`  (主板)
- `Chassis`  (机箱)
- `Processor`  (处理器)
- `Memory Controller`  (内存控制器)
- `Memory Module`  (内存模块)
- `Cache`  (高速缓存)
- `Port Connector`  (端口连接器)
- `System Slots`  (系统插槽)
- `On Board Devices`  (在线设备)
- `OEM Strings`  (OEM字符串)
- `System Configuration Options`  (系统配置选项)
- `BIOS Language`  (BIOS 语言)
- `Group Associations`  (集团协会)
- `System Event Log`  (系统事件日志)
- `Physical Memory Array`  (物理内存阵列)
- `Memory Device`  (记忆装置)
- `32-bit Memory Error`  (32位内存错误)
- `Memory Array Mapped Address`  (内存阵列映射地址)
- `Memory Device Mapped Address`  (内存设备映射地址)
- `Built-in Pointing Device`  (内置指针设备)
- `Portable Battery`  (便携式电池)
- `System Reset`  (系统重置)
- `Hardware Security`  (硬件安全)
- `System Power Controls`  (系统电源控制)
- `Voltage Probe`  (电压探头)
- `Cooling Device`  (冷却装置)
- `Temperature Probe`  (温度探头)
- `Electrical Current Probe`  (电流探头)
- `Out-of-band Remote Access`  (带外远程访问)
- `Boot Integrity Services`  (启动完整性服务)
- `System Boot`  (系统启动)
- `64-bit Memory Error`  (64位内存错误)
- `Management Device`  (管理设备)
- `Management Device Component`  (管理设备组件)
- `Management Device Threshold Data`  (管理设备阈值数据)
- `Memory Channel`  (记忆频道)
- `IPMI Device`  (IPMI设备)
- `Power Supply`  (电源)
- `Additional Information`  (附加信息)
- `Onboard Device`  (板载设备)

# 实例

- 查看服务器型号:

```
    [root@caoxl ~]# dmidecode | grep 'Product Name'
    	Product Name: Alibaba Cloud ECS
```

- 查主板的序列号:

```
    [root@caoxl ~]# dmidecode |grep 'Serial Number'
    	Serial Number: ff1e4090-65f1-4fba-882d-ab08dc1e9119
    	Serial Number: Not Specified
    	Serial Number: Not Specified
    	Serial Number: Not Specified
```

- 查看系统序列号:

```
    [root@caoxl ~]# dmidecode -s system-serial-number
    ff1e4090-65f1-4fba-882d-ab08dc1e9119
```

- 查看内存信息:

```
    [root@caoxl ~]# dmidecode -t memory
    # dmidecode 3.0
    Getting SMBIOS data from sysfs.
    SMBIOS 2.8 present.
    
    Handle 0x1000, DMI type 16, 23 bytes
    Physical Memory Array
    	Location: Other
    	Use: System Memory
    	Error Correction Type: Multi-bit ECC
    	Maximum Capacity: 1 GB
    	Error Information Handle: Not Provided
    	Number Of Devices: 1
    
    Handle 0x1100, DMI type 17, 40 bytes
    Memory Device
    	Array Handle: 0x1000
    	Error Information Handle: Not Provided
    	Total Width: Unknown
    	Data Width: Unknown
    	Size: 1024 MB
    	Form Factor: DIMM
    	Set: None
    	Locator: DIMM 0
    	Bank Locator: Not Specified
    	Type: RAM
    	Type Detail: Other
    	Speed: Unknown
    	Manufacturer: Alibaba Cloud
    	Serial Number: Not Specified
    	Asset Tag: Not Specified
    	Part Number: Not Specified
    	Rank: Unknown
    	Configured Clock Speed: Unknown
    	Minimum Voltage: Unknown
    	Maximum Voltage: Unknown
    	Configured Voltage: Unknown
```

- 查看OEM信息:

```
    [root@caoxl ~]# dmidecode -t 11
    # dmidecode 3.0
    Getting SMBIOS data from sysfs.
    SMBIOS 2.8 present.
```

不带选项执行`dmidecode`命令通常会输出所有的硬件信息。`dmidecode`命令有个很有用的选项`-t`，可以按指定类型输出相关信息，假如要获得处理器方面的信息，则可以执行：

```
    [root@caoxl ~]# dmidecode -t processor
    # dmidecode 3.0
    Getting SMBIOS data from sysfs.
    SMBIOS 2.8 present.
    
    Handle 0x0400, DMI type 4, 42 bytes
    Processor Information
    	Socket Designation: CPU 0
    	Type: Central Processor
    	Family: Other
    	Manufacturer: Alibaba Cloud
    	ID: F1 06 04 00 FF FB 8B 0F
    	Version: pc-i440fx-2.1
    	Voltage: Unknown
    	External Clock: Unknown
    	Max Speed: Unknown
    	Current Speed: Unknown
    	Status: Populated, Enabled
    	Upgrade: Other
    	L1 Cache Handle: Not Provided
    	L2 Cache Handle: Not Provided
    	L3 Cache Handle: Not Provided
    	Serial Number: Not Specified
    	Asset Tag: Not Specified
    	Part Number: Not Specified
    	Core Count: 1
    	Core Enabled: 1
    	Thread Count: 1
    	Characteristics: None
```