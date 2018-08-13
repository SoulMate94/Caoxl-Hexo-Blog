---
title: Crontab Is So Easy
date: 2018-01-24 16:12:57
categories: Linux
tags: [Linx, Crontab]
---

> Crontab是一个Unix/Linux系统下的常用的定时执行工具，可以在无需人工干预的情况下运行指定作业。

<!--more-->

# 安装

## CentOS下 安装Crontab

```
    yum install vixie-cron crontabs      //安装Crontab
    
    chkconfig crond on                  //设为开机自启动
    service crond start                 //启动
```

**说明：** `vixie-cron`软件包是`cron`的主程序；`crontabs`软件包是用来安装、卸装、 或列举用来驱动 `cron` 守护进程的表格的程序。

## Debian下 安装Crontab

```
    apt-get install cron             //大部分情况下Debian都已安装。
    
    /etc/init.d/cron restart        //重启Crontab
```

# 使用方法

## 查看crontab定时执行任务列表

```
    crontab -l
```

## 添加crontab定时执行任务

```
    crontab -e
```

输入crontab任务命令时可能会因为crontab默认编辑器的不同。

![](https://www.vpser.net/uploads/2011/05/crontab-e-nano.jpg)

如上图所示为 [nano编辑器](https://www.vpser.net/manage/nano.html)，使用比较简单，直接在文件末尾按crontab命令格式输入即可，Ctrl+x退出，再输y 回车保存。

另外一种是`vi编辑器`，首先按i键，在文件末尾按crontab命令格式输入，再按ESC键，再输入`:wq` 回车即可。

## crontab 任务命令书写格式

```
格式：	minute	hour	dayofmonth	month	dayofweek	command
解释：	分钟	小时	日期	月付	周	命令
范围：	0-59	0～23	1～31	1～12	0～7，0和7都代表周日
```

在crontab中我们会经常用到`* ,   -  /n `这4个符号，好吧还是再画个表格，更清楚些：

符号|解释|
:----|:--
`*(星号)`|代表所有有效的值。 如：0 23 * * * backup 不论几月几日周几的23点整都执行backup命令。|
`,(逗号)`|代表分割开多个值。如：30 9 1,16,20 * * command 每月的1、16、20号9点30分执行command命令。|
`-(减号)`|代表一段时间范围。如0 9-17 * * * checkmail 每天9点到17点的整点执行checkmail命令|
`/n` |代表每隔n长时间。如 */5 * * * * check 每隔5分钟执行一次check命令，与0-59/5一样。|

**下面举一些例子来加深理解：**

```
    每天凌晨3:00执行备份程序：0 3 * * * /root/backup.sh
    
    每周日8点30分执行日志清理程序：30 8 * * 7 /root/clear.sh
    
    每周1周5 0点整执行test程序：0 0 * * 1,5 test
    
    每年的5月12日14点执行wenchuan程序：0 14 12 5 * /root/wenchuan
    
    每晚18点到23点每15分钟重启一次php-fpm：*/15 18-23 * * * /etc/init.d/php-fpm
```

# 参考

- [Linux VPS/服务器上用Crontab来定时执行实现VPS自动化](https://www.vpser.net/manage/crontab.html)
- [nano编辑器使用教程](https://www.vpser.net/manage/nano.html)