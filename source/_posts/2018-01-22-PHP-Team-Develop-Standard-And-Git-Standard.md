---
title: PHP 团队开发规范和 Git 使用规范
date: 2018-01-22 16:05:45
categories: PHP
tags: [PHP, Git, 规范]
---

# PHP 开发规范

## 为什么？

> 因为每个程序员都有自己的写代码的习惯, 很不利于团队开发,所以，对一个团队而言之所以需要这么一个规范主要是为了在软件开发过程中大家便于交流和协作。

**简单来说:就是让一群代码风格不一样的人,能互相明白对方在写些什么鬼~~~**

<!--more-->

关于我们要采用的 PHP 开发规范，是采用的 PHP 互操作性框架制定小组（FIG）制定的 [PHP-PSR](https://github.com/PizzaLiu/PHP-FIG) ，也就是传说中的 [PHP 标准规范](https://psr.phphub.org/)

下面我要讲的，就是在这个的基础上简单总结出来的要点。主要有下面这些：

## 程序文件层面

### 文件编码

可以在各编程语言间通用的。`UTF-8`、`不带字节序标记（BOM）`。因为 字节序标记 会阻止应用程序设置它的头信息，从而影响 PHP 的输出。

### 换行符

使用 Unix 格式换行符（LF）。

> 因为 PHP 基本上都是跑在 Linux 上的（其实大部分网络服务都是跑在 Linux 上的），LAMP，LNMP 这些被广泛采用的流行 Web 架构都是运行在 Linux 服务器上，而 Linux 衍生于 UNIX，所以使用 UNIX 换行符能从脚本文件层面上杜绝不必要的麻烦

**麻烦在于:**

Windows 换行使用的是 CR＋LF（回车和换行），UNIX/Linux 使用的是 LF，Mac OS X 使用的是 CR，

虽然大部分现代编辑器和 IDE 都具有自动转换这种差异的功能，但是如果被不小心弄错，导致本地开发的换行格式和 Linux 服务器上的不一致，比如在 Windows 上开发好后没问题，但是一放到 Linux 服务器上后却不能正常运行，这很可能就是因为 Windows 下的文件到 Linux 上后每次换行的地方都多了一个回车符号被显式地，以字符串 \M 的形式输出到原本的 PHP 脚本中，而这个 \M 是 PHP 解析器无法理解的语言结构，因此导致解析错误。


### 用空行代替 PHP 结束标签

PHP 结束标签 `?>` 对于 PHP 解析器来说是可选的，但是只要使用了，结束标签之后的空格有可能会导致不想要的输出，这个空格可能是由开发者或者用户又或者 FTP 应用程序引入的，甚至可能导致出现 PHP 错误，就算通过修改配置抑制输出 PHP 错误，仍然会出现空白页面。基于这个原因， **所有的 PHP 文件将不使用结束标签，而是以一个空行代替**。

### 文件命名

类名必须以大写开头的驼峰法命名，比如：`SiteController.php`，或带 .class 的下划线加小写命名，比如：`site_controller.class.php`。但是，**同一个项目中，只能选用其一**（下面所有可以有2种选择的都必须这样）。

默认推荐第一种，因为可以少敲几次键盘。

其他文件，比如视图、配置、一般的脚本文件等等只能使用小写加下划线命名，比如：home.html，admin_login.js，article_footer.css，site_config.php 等等。

### 文件夹和前缀

同一个逻辑下面的程序文件放在同一个有意义名称的文件夹下，如果不得不放在一个文件夹，那么使用前缀来区分。

比如：system 控制器下的所有视图必须放在 views 文件夹下的 system 目录下。如果非要放在 views 文件夹下，那么必须要带上 `system_` 前缀。默认采用第一种，因为可以缩短文件名。

## 代码层面

### 命名

> 所有命名都要 **见名知意**！

- **类名**必须大写是开头的驼峰法（匈牙利命名法），比如：`SiteController`。
- **常量名**必须是全大写，并且单词之间使用下划线，比如：`APP_ROOT`。
- **函数/方法名**必须是小写打头的驼峰法或下划线＋小写，比如：`getVersion()` 和 `get_version()`。默认使用第一种，因为为了和 PHP 系统函数区分。
- **变量/属性名**必须使用 $ 打头的下划线＋小写，比如：`$order_by`， `$is_same_key`，或者 驼峰法：`$orderBy`，`$OrderBy`。

> 上面含有多种可选方式的规范中，无论遵循哪种命名方式，都应该在一定的范围内保持一致。最好是同一个项目中只使用默认的一套，函数名和变量名不能使用同一套。

### 注释

> 在写代码的那一刻，只有程序员和上帝能看懂，写完之后，如果没有注释，就只有上帝能看懂了。

- 函数或方法名的注释必须在函数定义的上方，使用多行注释风格，每个方法必须要写注释。形如：

```
    /**
     * what this function do?
     * @para1 Type, like String
     * @para2 Type, like Boolean
     * @return Type, like Array
     * by @caoxl
       */
       
    public static function getVersion(&$arr, $flag=false)
    {
     # step 1
     ...    // what this variable means?
     
     # step 2
     ...    // what should be noticed here?
     
     # step 3
     if ( $flag ) {
     	...    // what does this line do ?	
     } elseif ( $cond ) {
     	...    // do something here
     }
    }
```

关于上面的示例代码，有几点需要说明的：

- 过程/流程注释使用 `#` 号，因为 `#` 有代表数字顺序都含义，所以适合描述步骤。
- 每行代码的注释写在同一行的右边，使用双斜线，上下行对齐。


## 关键字使用

- PHP所有 关键字必须全部小写。包括常量关键字 `true` 、`false` 和 `null` 也必须全部小写。
- 所有类方法都必须添加`访问修饰符`。
- 需要添加 `abstract` 或 `final` 声明时， 必须写在访问修饰符前，而 `static` 则必须写在其后。 比如：

```
  <?php
  
  namespace Vendor\Package ;
  
  use OtherVendor\OtherPackage\OtherClass ;
  
  class Site
  {
    abstract protected function drink() ;
      
    /**
    * what this function do ?
    */
         final public static function getVersion()
    {
      // do sth here
    } 
  }
```

## 空格、行和花括号

- 控制结构的开始花括号({)必须写在声明的同一行，而结束花括号(})必须写在主体后自成一行。
- 类和方法的起始花括号都要自成一行。
- 使用逗号的地方，逗号前不能有空格而逗号后必须有空格。
- 函数名与紧跟的括号之际要有空格，函数括号与相邻的参数不能有空格。
- 代码必须使用4个空格符的缩进，一定不能用 tab键 。
  > 因为不同的编辑器、IDE对 tab 键的理解不一样，有的是 4 个空格有的是8个空格，严格使用空格缩进可以避免这种差异带来的排版混乱。
- 每行不应该多于80个字符，大于80字符的行应该折成多行。
- 非空行后一定不能有多余的空格符。
- 每行一定不能存在多于一条语句。


## 其他方面

> 新项目默认选择最新稳定版。

- 开发工具：PHPStrom + Sublime Text/Atom。
- PHP 版本：5.5+ (根据项目要求: 如江湖外卖要求 5.4-nts版本, Lumen API要求 7.1+)
- MySQL版本: 5.5+ (同上: 如Lumen API要求 5.7+)

最后，用一句话总结关于开发规范的话题：

> **统一开发规范的价值在于我们都遵循这个编码风格，而不是在于它本身。**


# Git 项目管理规范

Git 工作流程大概有三种：
 - Git 工作流（含功能分支，补丁分支，和预发分支）
 - GitHub 工作流（只有一个长期分支）
 - Gitlab 工作流（上游优先；区分开发环境，预发环境和生产环境）。
 
Developer 将涉及到的 Git 操作将会有：

- 拉取项目源码到本地

```
    git clone git@git.keensword.net:hp-php/hpmall.git
```

- 在 dev 分支基础上创建新分支

```
    git checkout dev    # 确保在 dev 分支上
    git checkout -b taskname-developername
```

- 开发完成提交任务代码

```
    # 1. 添加
    git add -A
    # // 或者部分: git add file1 file2 ...
    
    # 2. 提交
    git commit -m "comments for this commit that make sense" 
    # // 或者使用 git commit 进入 vim 编辑器书写详细注释
    
    # 3. 拉取最新 dev 分支代码并合并
    git fetch origin dev   # 若使用的是别名请把 origin 改为你设置的别名
    git merge origin/dev   # 若出现冲突请与其他开发者协商解决
    # // 也可以用 git pull origin dev 一步到位但是不推荐因为当代码出错很难 debug
    
    # 4. 查看当前是否存在没提交的文件
    git status
    # // 如果有则重复 1 和 2 然后到 5
    
    # 5. 确认无冲突后提交到 gitlab
    git push origin taskname-developername
```
 
- 在 Gitlab 上发起 Merge Request

Source Branch 一定是自己的任务分支，Target Branch 一定是项目的 dev 分支。

- 一次任务完成后删除上次创建的本地任务分支

```
    git branch -d taskname-developername
```

**每次做任务的时候建议都删除旧的任务分支然后新建新任务分支。**


## 项目负责人将涉及的额外工作

- 决定是否接受开发者的合并请求、拒绝未与 dev 最新代码合并的合并请求。
- 定期将 dev 分支上的最新代码给上面组长或技术总监 review，如果都满意后将合并到 master 分支。
- 将 master 分支的代码部署、并定期更新到服务器。


# 团队建设

所以简单总结了以下几句话：

> 提交效率，**减少加班**。
> 
> 加强沟通，取长补短。
> 
> 研究技术，共同进步。

# 参考

- [PHP-The Right Way](http://www.phptherightway.com/)
- [PHP 之道](https://laravel-china.github.io/php-the-right-way/)
- [PHP 标准规范](https://psr.phphub.org/)
- [Git 使用规范流程](http://www.ruanyifeng.com/blog/2015/08/git-use-process.html)
- [Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)
- [常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)

## Book

- [《Clean Code》](http://blog.csdn.net/ljheee/article/details/52619547)