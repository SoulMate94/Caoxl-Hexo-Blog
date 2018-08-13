---
title: Linux 命令 「pstree」
date: 2018-07-24 14:57:39
categories: Linux cmd
tags: [Linux]
---

> **pstree命令**以树状图的方式展现进程之间的派生关系，显示效果比较直观。

<!-- more -->

# 语法

```
    pstree (选项)
```

# 选项

- `-a`: 显示每个程序的完整指令, 包含路径, 参数或是常驻服务的标示
- `-c`: 不使用精简标示法
- `-G`: 使用VT100终端机的列绘图字符
- `-h`: 列出树状图时, 特别标明现在执行的程序
- `-H`<程序识别码>: 此参数的效果和指定"-h"参数类似,但特别标明指定的程序
- `-l`: 采用长列格式显示树状图
- `-n`: 用程序识别码排序, 预设是以程序名称来排序
- `-p`: 显示程序识别码
- `-u`: 显示用户名称
- `-U`: 使用`UTF-8`列绘图字符
- `-V`: 显示版本信息

# 实例

- `pstree`

```
    [root@caoxianliang test]# pstree
    systemd─┬─AliYunDun───15*[{AliYunDun}]
            ├─AliYunDunUpdate───3*[{AliYunDunUpdate}]
            ├─agetty
            ├─aliyun-service───6*[{aliyun-service}]
            ├─atd
            ├─auditd───{auditd}
            ├─crond
            ├─dbus-daemon
            ├─mysqld_safe───mysqld───15*[{mysqld}]
            ├─nginx───nginx
            ├─ntpd
            ├─php-fpm───2*[php-fpm]
            ├─polkitd───5*[{polkitd}]
            ├─python
            ├─rsyslogd───2*[{rsyslogd}]
            ├─screen───bash
            ├─sshd───sshd───bash───pstree
            ├─systemd-journal
            ├─systemd-logind
            ├─systemd-udevd
            └─tuned───4*[{tuned}]
```

- 显示当前所有进程的进程号和进程id

```
    [root@caoxianliang test]# pstree -p
    systemd(1)─┬─AliYunDun(17050)─┬─{AliYunDun}(17051)
               │                  ├─{AliYunDun}(17052)
               │                  ├─{AliYunDun}(17054)
               │                  ├─{AliYunDun}(17055)
               │                  ├─{AliYunDun}(17056)
               │                  ├─{AliYunDun}(17057)
               │                  ├─{AliYunDun}(17058)
               │                  ├─{AliYunDun}(17059)
               │                  ├─{AliYunDun}(17060)
               │                  ├─{AliYunDun}(17061)
               │                  ├─{AliYunDun}(17062)
               │                  ├─{AliYunDun}(17063)
               │                  ├─{AliYunDun}(17064)
               │                  ├─{AliYunDun}(17072)
               │                  └─{AliYunDun}(28696)
               ├─AliYunDunUpdate(787)─┬─{AliYunDunUpdate}(789)
               │                      ├─{AliYunDunUpdate}(790)
               │                      └─{AliYunDunUpdate}(797)
               ├─agetty(487)
               ├─aliyun-service(1475)─┬─{aliyun-service}(1478)
               │                      ├─{aliyun-service}(1479)
               │                      ├─{aliyun-service}(1480)
               │                      ├─{aliyun-service}(1481)
               │                      ├─{aliyun-service}(1482)
               │                      └─{aliyun-service}(1483)
               ├─atd(480)
               ├─auditd(433)───{auditd}(434)
               ├─crond(481)
               ├─dbus-daemon(458)
               ├─mysqld_safe(12935)───mysqld(13453)─┬─{mysqld}(13455)
               │                                    ├─{mysqld}(13456)
               │                                    ├─{mysqld}(13457)
               │                                    ├─{mysqld}(13458)
               │                                    ├─{mysqld}(13459)
               │                                    ├─{mysqld}(13460)
               │                                    ├─{mysqld}(13461)
               │                                    ├─{mysqld}(13462)
               │                                    ├─{mysqld}(13463)
               │                                    ├─{mysqld}(13464)
               │                                    ├─{mysqld}(13466)
               │                                    ├─{mysqld}(13467)
               │                                    ├─{mysqld}(13468)
               │                                    ├─{mysqld}(13469)
               │                                    └─{mysqld}(13472)
               ├─nginx(17121)───nginx(17122)
               ├─ntpd(461)
               ├─php-fpm(1352)─┬─php-fpm(1353)
               │               └─php-fpm(1354)
               ├─polkitd(456)─┬─{polkitd}(464)
               │              ├─{polkitd}(465)
               │              ├─{polkitd}(473)
               │              ├─{polkitd}(475)
               │              └─{polkitd}(478)
               ├─python(1175)
               ├─rsyslogd(712)─┬─{rsyslogd}(750)
               │               └─{rsyslogd}(1383)
               ├─screen(1697)───bash(1698)
               ├─sshd(1444)───sshd(1685)───bash(1688)───pstree(2250)
               ├─systemd-journal(319)
               ├─systemd-logind(470)
               ├─systemd-udevd(342)
               └─tuned(704)─┬─{tuned}(1395)
                            ├─{tuned}(1396)
                            ├─{tuned}(1397)
                            └─{tuned}(1410)
```

- 显示所有进程的所有详细信息，遇到相同的进程名可以压缩显示。

```
    [root@caoxianliang test]# pstree -a
    systemd --switched-root --system --deserialize 22
      ├─AliYunDun
      │   └─15*[{AliYunDun}]
      ├─AliYunDunUpdate
      │   └─3*[{AliYunDunUpdate}]
      ├─agetty --noclear tty1 linux
      ├─aliyun-service
      │   └─6*[{aliyun-service}]
      ├─atd -f
      ├─auditd
      │   └─{auditd}
      ├─crond -n
      ├─dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
      ├─mysqld_safe /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/var ...
      │   └─mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var--plugin-dir=/usr/local/mysql/lib/plugi
      │       └─15*[{mysqld}]
      ├─nginx
      │   └─nginx                                          
      ├─ntpd -u ntp:ntp -g
      ├─php-fpm                                                                   
      │   ├─php-fpm                                                                                                          ...
      │   └─php-fpm                                                                                                          ...
      ├─polkitd --no-debug
      │   └─5*[{polkitd}]
      ├─python /usr/local/shadowsocks/server.py -c /etc/shadowsocks.json -d start
      ├─rsyslogd -n
      │   └─2*[{rsyslogd}]
      ├─screen
      │   └─bash
      ├─sshd -D
      │   └─sshd    
      │       └─bash
      │           └─pstree -a
      ├─systemd-journal
      ├─systemd-logind
      ├─systemd-udevd
      └─tuned -Es /usr/sbin/tuned -l -P
          └─4*[{tuned}]
```