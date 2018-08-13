---
title: Shell 脚本笔记
date: 2018-07-04 15:04:32
categories: Linux
tags: [Linux, Shell]
---

> `sh`脚本结合系统命令便有了强大的威力，在字符处理领域，有`grep`、`awk`、`sed`三剑客，
- <u>[grep](http://blog.caoxl.com/2018/06/28/Linux-cmd-grep/)</u>:负责找出特定的行
- `awk`能将行拆分成多个字段
- `sed`则可以实现更新插入删除等写操作。

<!-- more -->

# 几种常见的 `Shell`

## `bash`

> `bash` 是 `Linux` 标准默认的 `shell`, 是`BourneAgain Shell`的缩写


## `sh`

`sh` 由 `Steve Bourne` 开发，是 `Bourne Shell` 的缩写，`sh` 是 `Unix` 标准默认的 `shell`。


## `ash`

`ash shell` 是由 `Kenneth Almquist` 编写的，`Linux` 中**占用系统资源最少**的一个小 `shell`，它只包含`24`个内部命令，因而使用起来很不方便。

## `csh`

`csh` 是 `Linux` 比较大的内核，它由以 `William Joy` 为代表的共计`47`位作者编成，共有`52`个内部命令。
该 `shell` 其实是指向 `/bin/tcsh` 这样的一个 `shell`，也就是说，**csh 其实就是 tcsh**。

## `ksh`

`ksh` 是 `Korn shell` 的缩写，由 `Eric Gisin` 编写，共有`42`条内部命令。
该 `shell` 最大的优点是几乎和商业发行版的 `ksh` 完全兼容，这样就可以在不用花钱购买商业版本的情况下尝试商业版本的性能了。

# 实例

## 定时检测`nginx`、`mysql`是否被关闭

```shell
    path=/var/log
    log=${path}/httpd-mysql.log
    
    name=(apache mysql)
    
    exs_init[0]="service httpd start"
    exs_init[1]="/etc/init.d/mysqld restart"
    
    for ((i=0; i<2; i++)); do
        echo "检查${name[i]}进程是否存在"
        ps -ef | grep ${name[i]} | grep -v grep
        if [ $? -eq 0]; then
            pid=$(pgrep -f ${name[i]})
            echo "`date +"%Y-%m-%d %H:%M:%S"` ${name[$i]} is running with pid $pid" >> ${log}
        else
            $(${exs_init[i]})
            echo "`date +"%Y-%m-%d %H:%M:%S"` ${name[$i]} start success" >> ${log}
        fi
    done   
```

> **说明:** 检测 `nginx`、`mysql`进程是否存在，如果不存在了会自动重新启动。 脚本每次运行会写日志的，没事可以去看看该日志文件，
如果进程是不是真的经常性不存在，恐怕就要排查一下深层原因了。

```linux
    [root@izj6c6djex81rijczh0t8yz test]# sh check_nginx.sh 
    检查apache进程是否存在
    Redirecting to /bin/systemctl start httpd.service
    Failed to start httpd.service: Unit not found.
    检查mysql进程是否存在
    root       755     1  0 Jun25 ?        00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/var --pid-file=/usr/local/mysql/var/izj6c6djex81rijczh0t8yz.pid
    mysql     1349   755  0 Jun25 ?        00:03:04 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/var --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/usr/local/mysql/var/izj6c6djex81rijczh0t8yz.err --open-files-limit=65535 --pid-file=/usr/local/mysql/var/izj6c6djex81rijczh0t8yz.pid --socket=/tmp/mysql.sock --port=3306
```

# 开始学习 `Shell` 啦~~

## `Shell` 变量

- 定义变量

定义变量时，变量名不加美元符号（$），如：

```
    variableName="value"
```

- 使用变量

```
    your_name="caoxl"
    echo $your_name
```

- 重新定义变量

```
    my_url="https://caoxl.com/shell"
    echo ${my_url}
    my_url="https://caoxl.com/linux"
    echo ${my_url}
```

- 只读变量

```
    my_url="https://caoxl.com/shell"
    readonly my_url
    my_url="https://caoxl.com/linux"
```

运行结果:

```
    /bin/sh:    NAME:This variable is read only.
```

- 删除变量

```
    unset variable_name
```

### 变量类型

- 局部变量
- 环境变量
- `shell`变量

## `Shell` 特殊变量

- `$0`: 当前脚本的文件名
- `$n`: 传递给脚本或函数的参数
- `$#`: 传递给脚本或函数的参数个数
- `$*`: 传递给脚本或函数的所有参数
- `$@`: 传递给脚本或函数的所有参数.被双引号(`" "`)包含时，与 `$*` 稍有不同
- `$?`: 上个命令的退出状态,或函数的返回值
- `$$`: 当前`Shell`进程`ID`,对于`Shell`脚本.就是这些脚本所在的进程ID

## `Shell` 替换

如果表达式中包含特殊字符，`Shell` 将会进行替换。
例如，在双引号中使用变量就是一种替换，转义字符也是一种替换。

- `\`:  反斜杠
- `\a`: 警报,响铃
- `\b`: 退格(删除键)
- `\f`: 换页(FF), 将当前位置移到下页开头
- `\n`: 换行
- `\r`: 回车
- `\t`: 水平制表符(tab键)
- `\v`: 垂直制表符

## 命令替换

> 命令替换是指 `Shell` 可以先执行命令，将输出结果暂时保存，在适当的地方输出。

命令替换的语法： "\`\`" 注意是反引号，不是单引号，这个键位于 Esc 键下方。

### 实例

```
    #!/bin/bash
    
    DATE=`date`
    echo "Date is $DATE"
    
    USERS=`who | wc -l`
    echo "Logged in user are $USERS"
    
    UP=`date ; uptime`
    echo "Uptime is $UP"
```

- 结果:

```
    Date is 
    Logged in user are 1
    Uptime is Wed Jul  4 17:20:55 CST 2018
     17:20:55 up 9 days, 18 min,  1 user,  load average: 0.00, 0.01, 0.05
```

### 变量替换

- `${var}`: 变量本来的值
- `${var:-word}`:   如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值
- `${var:=word}`:   如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。
- `${var:?message}`:    如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标 准错误输出，
可以用来检测变量 var 是否可以被正常赋值。若此替换出现在 Shell 脚本中，那么脚本将停止运行。
- `${var:+word}`:   如果变量 var 被定义，那么返回 word，但不改变 var 的值。

## `Shell` 运算符

- `expr`

> `expr` 是一款表达式计算工具，使用它能完成表达式的求值操作。

两点注意:
1. 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样
2. 完整的表达式要被"\`"包含，注意这个字符不是常用的单引号，在 Esc 键下边。

### 算术运算符

```
    #!/bin/sh
    
    a=10
    b=20
    val=`expr $a + $b`
    echo "a + b : $val"
    
    val=`expr $a - $b`
    echo "a - b : $val"
    
    val=`expr $a \* $b`
    echo "a * b : $val"
    
    val=`expr $b / $a`
    echo "b / a : $val"
    
    val=`expr $b % $a`
    echo "b % a : $val"
    
    if [ $a == $b ]
    then
       echo "a is equal to b"
    fi
    
    if [ $a != $b ]
    then
       echo "a is not equal to b"
    fi
```

注意:
1. 乘号(`*`)前边必须加反斜杠(\)才能实现乘法运算
2. if...then...fi 是条件语句，后续将会讲解。

### 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

- `-eq`:  检测两个数是否相等，相等返回 true。
- `-ne`:  检测两个数是否相等，不相等返回 true。
- `-gt`:  检测左边的数是否大于右边的，如果是，则返回 true。
- `-lt`:  检测左边的数是否小于右边的，如果是，则返回 true。
- `-ge`:  检测左边的数是否大等于右边的，如果是，则返回 true。
- `-le`:  检测左边的数是否小于等于右边的，如果是，则返回 true

### 布尔运算符

- `!`:  非运算，表达式为 true 则返回 false，否则返回 true。
- `-o`: 或运算，有一个表达式为 true 则返回 true。
- `-a`: 与运算，两个表达式都为 true 才返回 true。

### 字符串运算符

- `=`:  检测两个字符串是否相等，相等返回 true。
- `!=`: 检测两个字符串是否相等，不相等返回 true
- `-z`: 检测字符串长度是否为0，为0返回 true。
- `-n`: 检测字符串长度是否为0，不为0返回 true。
- `str`: 检测字符串是否为空，不为空返回 true。

### 文件测试运算符

> 文件测试运算符用于检测 `Unix` 文件的各种属性。

## `Shell` 注释

以`“#”`开头的行就是注释，会被解释器忽略。

`sh` 里**没有多行注释**，只能每一行加一个#号。只能像这样：

## `Shell` 字符串

字符串是 shell 编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），
**字符串可以用单引号，也可以用双引号，也可以不用引号**。单双引号的区别跟 PHP 类似。

### 单引号

单引号字符串的限制： 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的;
单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

### 双引号

双引号的优点： 双引号里可以有变量 双引号里可以出现转义字符

### 拼接字符串

```
    str1="hello"
    str2="shell"
    
    echo $str1 $str2
```

### 获取字符串长度

```
    str="abcd"
    echo ${#str} #输出4
```

### 提取子字符串

```
    str="alibaba is a great company"
    echo ${str:0:6} #输出 alibaba
```

### 查找子字符串

```
    str="alibaba is a great company"
    echo `expr index "$str" is`
```

## `Shell` 数组

## `Shell echo命令`

## `Shell printf命令`

## `Shell if else语句`

## `Shell case esac语句`

## `Shell for循环`

## `Shell while循环`

## `Shell until循环`

## `Shell` 跳出循环

## `Shell` 函数

## `Shell` 函数参数

## `Shell` 输入输出重定向

## `Shell` 文件包含