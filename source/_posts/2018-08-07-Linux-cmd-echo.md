---
title: Linux 命令 「echo」
date: 2018-08-07 09:13:47
categories: Linux cmd
tags: [Linux cmd]
---

> **echo命令**用于在shell中打印shell变量的值，或者直接输出指定的字符串。

<!-- more -->

越简单的东西, 越容易被忽视.

# 语法

```
    echo (选项) (参数)
```

# 选项

```
    -e: 激活转义字符
```

使用 `-e`选项时, 若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出:

- `\a`: 发出警告声
- `\b`: 删除前一个字符
- `\c`: 最后不加上换行符号
- `\f`: 换行但光标仍旧停留在原来的位置
- `\n`: 换行且光标移至行首
- `\r`: 光标移至行首, 但不换行
- `\t`: 插入tab
- `\v`: 与`\f`相同
- `\\`: 插入`\`字符
- `\nnn`: 插入`nnn`(八进制)所代表的ASCII字符

# 参数

- 变量: 指定要打印的变量

# 实例

## 用`echo`命令打印带有色彩的文字

### 文字色:

```linux
    [root@caoxl ~]# echo -e "\e[1;31mThis is red text\e[0m"
    This is red text
```

- `\e[1;31m`: 将颜色设置为红色
- `\e[0m`: 将颜色重新置回

> 颜色码：重置=0，黑色=30，红色=31，绿色=32，黄色=33，蓝色=34，洋红=35，青色=36，白色=37

**例如:**

```
    echo -e "\033[30m 黑色字 \033[0m"
    echo -e "\033[31m 红色字 \033[0m"
    echo -e "\033[32m 绿色字 \033[0m"
    echo -e "\033[33m 黄色字 \033[0m"
    echo -e "\033[34m 蓝色字 \033[0m"
    echo -e "\033[35m 紫色字 \033[0m"
    echo -e "\033[36m 天蓝字 \033[0m"
    echo -e "\033[37m 白色字 \033[0m"
```

### 背景色:

```
    [root@caoxl ~]# echo -e "\e[1;42mGreed Background\e[0m"
    Greed Background
```

> 颜色码：重置=0，黑色=40，红色=41，绿色=42，黄色=43，蓝色=44，洋红=45，青色=46，白色=47

**例如:**

```
   echo -e "\033[40;37m 黑底白字 \033[0m"
   echo -e "\033[41;37m 红底白字 \033[0m"
   echo -e "\033[42;37m 绿底白字 \033[0m"
   echo -e "\033[43;37m 黄底白字 \033[0m"
   echo -e "\033[44;37m 蓝底白字 \033[0m"
   echo -e "\033[45;37m 紫底白字 \033[0m"
   echo -e "\033[46;37m 天蓝底白字 \033[0m"
   echo -e "\033[47;30m 白底黑字 \033[0m" 
```

### 文字闪动:

```
    [root@caoxl ~]# echo -e "\033[37;31;5mMySQL Server Stop...\033[39;49;0m"
    MySQL Server Stop...
```

> 红色数字处还有其他数字参数：0 关闭所有属性、1 设置高亮度（加粗）、4 下划线、5 闪烁、7 反显、8 消隐

**控制选项说明:**

```
    \33[0m 关闭所有属性 
    \33[1m 设置高亮度 
    \33[4m 下划线 
    \33[5m 闪烁 
    \33[7m 反显 
    \33[8m 消隐 
    \33[30m -- \33[37m 设置前景色 
    \33[40m -- \33[47m 设置背景色 
    \33[nA 光标上移n行 
    \33[nB 光标下移n行 
    \33[nC 光标右移n行 
    \33[nD 光标左移n行 
    \33[y;xH设置光标位置 
    \33[2J 清屏 
    \33[K 清除从光标到行尾的内容 
    \33[s 保存光标位置 
    \33[u 恢复光标位置 
    \33[?25l 隐藏光标 
    \33[?25h 显示光标 
```

## 使用`\a`选项

> `-e`后面跟上`\a`选项会听到声音警告。

```
    [root@caoxl ~]# echo -e "Tecmint is a community of \aLinux Nerds" 
    Tecmint is a community of Linux Nerds
```
