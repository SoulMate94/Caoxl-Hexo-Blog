---
title: MySQL  基础知识
date: 2017-12-31 17:58:29
categories: MySQL
tags: [MySQL, Database]
---

# MySQL 基础知识

> 记录MYSQL学习以及开发中的一些基础知识点


<!--more-->


# 数据库基本概念

## 分类

### 关系型

现实实体关系保存在一张张二维表内,每张表内的结果和定义是完全相同的。

`表示记录的容器,数据库是表的容器`。

### NoSQL

数据结构不一致的非关系型数据库。常用键值对存储。

### 对象型/ORM

一般大型的框架里面都会封装 ORM,是为了解决数学模型和软件工程概念之间的矛盾而出现的一种中继数据层。

## 关系型数据库

### DB vs DBMS

数据库（存储数据的物理实体）和数据库管理软件（管理数据库数据的软件系统）是两种东西

### 关系型数据库（SQL）操作分类

#### DDL：数据库定义语言

用于定义数据库的三级结构,包括外模式、概念模式、内模式及其相互之间的映像,定义数据的完整性、安全控制等约束。

DDL不需要 commit。

```
    CREATE / ALTER / DROP / TRUNCATE / COMMENT / RENAME
```

#### DML：数据库操纵语言

由 DBMS 提供,用于让用户或程序员使用,实现对数据库中数据的操作。
DML 分成交互型 DML 和 嵌入型 DML两类。
依据语言的级别,DML又可分成过程性DML 和 非过程性DML两种。

需要 commit。

```
    SELECT / INSERT / UPDATE / DELETE / MERGE / CALL / EXPLAIN / PLAN / LOCK TABLE
```

#### DCL：数据库控制语言
     
授权,角色控制等。

```
    # 授权 / 取消授权
    GRANT / REVOKE
```
#### DTL：事务控制语言

```
    SAVEPOINT    -- 设置保存点
    ROLLBACK [TO SAVEPOINT]     -- 回滚
    START TRANSCATION
    COMMIT
```

## 设计和使用

### 如何设计数据库？

> 取决于业务经验多少。

### 如何设计一张表？

关键在于设计表存储的结构,分析实体有哪些属性已经这些属性要怎样去存储。

### 如何分析和设计实体间的关系？

实体间关系有哪些？主要分为以下三种：

#### 1 对 1

两张实体表存在结构相同的主键字段,并且记录的主键值等于另一个关系表内的主键值。

#### 1 对 n(n 对 1)

在多的那端增加一个字段,用于指向该实体所属的另一个实体的标识。

#### n 对 m

利用一个中间表来表示实体之间的对应关系,中间表的每一条记录,表示一个关系。
其实 M => N,可以分解成 M => 1, N => 1,其中那个 1 代表的就是中间表。

### 如何使用 MySQL 数据库？

> 取决于对 MySQL 软件语法了解多少。

# 校对规则

## 校对规则有何用？

在当前编码下字符之间的比较顺序关系（排序条件）。

每一套编码字符集都有与之相关的一组或多组校对规则,每套字符集都有一套默认的校对规则。

校对规则可以规定不同的比较顺序（排序方式）,从而按需显示。

*说明*

校对规则不会影响数据的保存格式,只有字符集才影响。

## 校对规则的命名规范

```
    字符集_地区名_比较规则
```

其中,`ci`,`cs`,`bin`,分别代表不区分大小写,区分大小写,字节比较。

# 存储引擎

## 定义

即表的数据类型，指的是数据表的存储机制，比如用什么数据结构进行管理，以及索引方案等配套的相关功能。

默认存储引擎可以在 my.ini 中修改 `default-storage-engine=INNODB`。

也可以通过以下命令指定表的存储引擎：

```
    alter table tb_name engine myisam;
    create table tb_name(...) engine myisam;
```

*注意*

从支持外键的存储引擎并且表内已经有外键的话，更换到不支持的存储引擎时这样修改不会成功。

## MyISAM 和 InnoDB

> 可能会随着版本升级而有所改变

### 不同的存储引擎的存储文件后缀，以及创建一个表事创建的文件个数是不一样的

如 InnoDB 创建一个表文件的后缀是 .frm。其为结构文件，InnoDB 的数据文件在data 目录下的名为 ibdata 字样的文件中，表空间内存储着表数据和表索引，一 个表一个文件。

而 MyISAM 创建一个表文件的后缀是 .myd（数据文件），一个表三个文件（还有 2 个是.frm , .myi => 索 引）

### MyISAM 比 InnoDB 快

由于 MyISAM 文件分开存储，而索引文件可以被压缩，而 InnoDB 中索引和数据是绑定的，不能被压缩，体积大，所以 MyISAM 查询速度比 InnoDB 快。

### 并发和查询

MyISAM 是 MySQL 5.5 之前默认的存储引擎，由于不支持事务所以读写速度较快，但是容易在进行写操作的时候进行全表加锁的问题，表级锁触发后，当对表有操作时整个表都不能被再同时修改，直到当前操作结束为止，这时候如果写频率很快就容易造成写阻塞，并发性表现低。

**MyISAM 擅长查询，插入和检索**。InnoDB 是行级锁，支持对表的多行操作，并发操作支持比较好。**InnoDB 擅长更新和删除**。

Innodb 是 5.5 之后默认的存储引擎，也是互联网项目中建议使用的存储引擎，在维护上和效率上都比较高效。

### select 时是否整表全扫
    
InnoDB 不保存表的具体行数，所以在执行 `select count(*) from table` 时，InnoDB 需要完整扫描一遍表，来计算有多少 行，而 MyISAM 只需读出保存好的行数即可。

但是在执行 `select count(*) from table where` ... 时两种引擎效率相同。


### 联合索引

对于 auto_increment 类型的字段， InnoDB 中必须包含只有该字段的索引，而 MyISAM 可以和其他字段一起建立联合索引

### 其他存储引擎

 - MRG_MYISAM
 
 基于 MyISAM，可以把多个结构相同的基于 MYISAM 的表进行合并成一 个表进行处理(类似视图和分区)
 
 - Archive
 
 更趋向于存储日志类数据，存储容量相对较小。
 
 - Ndb cluster
 
 MySQL 集群中使用的。（内存型，如果数据大小大于内存则不使用该引擎）
 
### 选择存储引擎的原则

 - 性能
 
 - 功能


# 数据类型

## 字段类型和列类型

MySQL 使用字段保存数据的，而字段的结构是列。列的类型指的就是具体的「值」。

## 如何选择列类型？

 - 尽量精确
 - 优先考虑应用语言的处理要求而不是先考虑对 MySQL 数据库怎么最好
 - 考虑兼容性，数据库内数据的迁移问题
 
## 数值

### 整数型

采用补码存储，不取决于具体语言，而取决于计算机指令集结构。

 - **tinyint** => 1 Byte => -128~127 | 0~255
 - **smallint** => 2 Bytes => -32768~32767 | 0~65535
 - **mediumint** => 3 Bytes => -8388608~8388607 | 0~16777215
 - **int** => 4 Bytes => -2147483648~2147483647 | 0~4294967295
 - **bigint** => 8 Bytes => -9223372036854775808~9223372036854775807|0~18446744073709551615
 

### 浮点型

永远会丢失精度。

 - **float** => 单精度 => 4 Bytes，默认精度 6 位左右
 - **double** => 双精度 => 8 Bytes，默认精度 16 位左右
 - **decimal** => 定点数（精度为指定） => 可变长存储，大致是每 9 个数字采用 4 字节存储；整数和小数分开算
 
#### 浮点数科学计数法

```
    1.2e3 => 1200
    12.E3 => 12000
```

### 说明

 - MySQL 没有布尔类型。通常使用 tinyint(1) 来替代。
 - 当超过类型范围是会报错而不会像 `PHP` 那样自动类型转换。
 - **什么叫“浮点”？如何控制浮点数的数值范围？**
 小数点可以移动。控制数值范围的形式为 `type(M,D)`，其中 M 表示总位数，D 表示其中的小数点位数，因此整数位数=M-D。
 
 当浮点数精度丢失不会报错。当小数点超过规定的值时不会报错，只是输出还是规定的位数，而整数位超出会报错。
 
 - **关于浮点数的「无符号」**
 浮点数虽然也支持无符号，但它的无符号只是从逻辑上告诉不能存储负数而已，而不能向整型一样改变小数所能表示的值的范围。（显示范围受幂的影响）
 
 - **什么叫定点数？有什么特点？定点数是怎么保存数据的？怎么用？” 变长 “代表什么意思？**
 
 decimal，属于可变长不丢失精度的存储方式，是将数的所有位数都保存在计算机硬盘上，因此不会出现位数的丢失。通 常情况下，小数整数分开算，每 9 位 4 个字节存储。但也会根据具体情况优化，用于保存的位数够用就行，减少浪费这就是 “变长” 的含义。
 
 `decimal(M,D)`，M 是最大总位数，D 依然是最大小数位数。默认情况，M 为 10，D 为 0。
 
 **当小数超过规定位数时 decimal 会自动四舍五入。而整数只允许在最大范围内而不能四舍五入。**
 
 - 手机号型数据选择 `char` 还是 `int`？
 
 用 char 表示，因为 int 整型主要是可以用于计算的，手机号虽然是数字但不用于运算操作，所以其本质也相当于字符串。
 
 如果为了性能优先，又能确保手机号全是数字，则推荐使用 bigint 型。
 

### 时间日期

 - 类型 => 显示格式 => 取值范围 => 存储空间
 - **datetime** => YYYY-MM-DD HH:MM:SS => 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59 => 8 bytes
 - **timestamp** => YYYY-MM-DD HH:MM:SS => 1970-01-01 00:00:00 ~ 2038-01-19 03:14:07 => 4 bytes
 - **date** => YYYY-MM-DD => 1000-01-01~ 9999-12-31 => 3 bytes
 - **time** => HH:MM:SS => -838:59:59~ 838-59-59 => 3 bytes
 - **year** => YYYY => 1901~2155 => 1 byte
 
#### 关于 timestamp，从 DBA 角度从 PHP 开发者的角度使用时间类型有什么区别?

以整型存储(同 PHP 里时间戳的表示)，但表现形式依然是日期类型，截至日期为什么是那么多，是因为 4 个字节保存的范围只有那么多。

如果是 MySQL 的数据库管理员，则需要使用上面列出的所有时间日期类型，但是从一个 PHP 程序员的角度来看，最常用的是 PHP 自己的时间戳（UNIX 时间戳）而不是 MySQL 的时间戳，即直接用整形的形式将时间保存在数据库内。

因为整形保存数据基本上每种语言都支持，从这个角度讲，使用通用性比较好的 UNIX 时间戳，程序的兼容性将比较 好，也容易移植和升级。

#### 关于分隔符

支持任意形式的分隔符甚至不要分隔符，MySQL 是以范围自动区分，所以只要范围未溢出并且不太影响事实都能正常输 出，但最好是按标准输出不要总是使用特殊字符，因为有时候会造成误解。

比如只输入 2 位数表示日期时必须要明确 0-69 指的是 20xx 年，70-99 指的是 19xx 年，70-69 指的是 1970-2069 才能减少误解程度：

```
    insert into tb_name values(
      '14:28:30',
      '2015+04+21',
      '2015*04*21*14*57*30',
      '1994@03@09@00@00@01',
      2015
    );
```

#### 关于 0 值

如 2015-5-0 代表的就是 5 月整个月。


#### 关于 time

time 类型除了表示一天的时间外还可以表示时间间隔 “多少天多少小时多少分多少秒”，可用于表示过去了多久。

格式为：D HH:MM:SS。D 表示天数，范围大概 35 天内。比如：

```
    insert into tb_name cloumn_name values('8 5:23:56');    -- 表示 8 天 5 小时 23 分 56 秒
```
 
## 字符串

 - 类型 => 最大长度 => 备注
 - **char**    => 255 => `char(M)`（M 表示字符数）
 - **varchar** => 65535，但需要 1～2 位来保存信息，同时由于记录的限制，因此最大为 65532 => 可存储的字符个数根据编码不同而不同：
   - gbk => 0~32767;
   - utf8 => 0~21845
 - **tinytext/text/mediumtext/longtext** => L-1（L 为最大长度）;分别为：2^8-1/2^16-1/2^24-1/2^32-1 => 定义时通常不用手动指定长度，MySQL 可以自动计算
 - **enum**    => 1、2；枚举选项量：65535 => 内部存储也是整型 字段只能是枚举值中其一（单选）
 - **set**     => 1、2、3、4、8；元素数量：64
 - **binary/varbinary/blob** => char/varchar/text 类似 => 是存储二进制数据（字节而非字符）
 
### char(M) 和 varchar(M) 的区别？

M 表示的是允许的字符串长度，注意不是字节数，但是总长度的使用还是按照字节来计算的，即存储的时候还是按照字节数来存储。这将导致不同字符集的字符采用相同的 char 或 varchar 占用的字 节数将不一样，反之也成立。

在 char 中 M 是严格固定的长度，在 varchar 中 M 是可变长的（即 var 代表的其实是 variable），表示的是允许的最大长度。这样的特点导致了 char 的执行速率将会比 varchar 要高一点，但是灵活性不如 varchar，因为 varchar 可以事先计算出字符串的长度，从而灵活分配存储空间，避免空间的浪费。

但这灵活性是建立在额外运算时间和额外的一个字节保存字符串长度为代价的。

### varchar 的存储原理

总长度 65535 ，当类型数据超过 255 个字符时，用 2 个字节来保存长度。此情况下如果字符可以为空，则还需要 1 个字节保存字符是否为空这个信息，即此时能用于保存字符的字节数为 65535-2-1=65532。

如果不能为空 (not null)，而且是表整个记录的所有的字段同时都非空，才不需要额外的一个字节来保存空信息，即这个字节的作用是保存所有字段是否为空，即此时能用于保存字符的字节数为 65535-2=65533。

### text 和 varchar

text 所有字节都能用于保存数据。

### enum 和 set

单选和多选(集合)的区别。对数据库而言可以节省空间，但对于 PHP 项目不一定最适用。

```
    create table tb_name (
    	gender enum('female','male')
    );
    -- enum 起始值为 1 而不是 0
    insert into tb_name values('male'); <=> insert into tb_name values(2);
    select gender+0 from tb_name;    -- 该查询结果根据最后一次 insert 操作动态变化
```

enum 支持 +0 检索。set 的 +0 检索做的是位运算，是对所选的几个选项顺序号的二进制表示相加。

```
    create table tb_name (
    	lang set('c','php','python','java')
    );
    -- 错误示例
    insert into tb_name values('c, php,python');    -- !!! 两个元素之间不能含有空格等其他符号
    -- 正确示例
    insert into tb_name values('c,php');
    select lang+0 from set_test;    -- 该查询结果根据最后一次 insert 操作动态变化
```

### binary（字节与）

二进制数据没有编码的概念，始终都是字节而不能有效地转化为字符。

字符是有各种编码方案的，虽然在最底层都是二进制数据但在应用层时字符却能被编码而以人能理解的方式被解码输出。

数据库虽然可以存图片但是基本上没有这 么做，几乎都是保存二进制文件的文件路径，然后通过路径利用文件系统或浏览器去获取图片。但有些很特殊的情况如果确实需要保存二进制数据则 MySQL 也支持。


## 特殊类型

 - NULL 站在数据的角度怎么理解？
 
 NULL 也代表一个数据，也会占有数据空间，因为它代表“什么都没有”这个信息，本身占空间。


## 关于数据类型的长度

 - CHAR、VARCAHR 的长度是指字符的长度，即字符串个数
 
 例如 char(3) 则只能放字符串 “123”，如果插入数据 “1234”，则从高位截取，变为 “123”。
 
 注意这里的字符串个数无论是什么语言都一视同仁，包括中文，即 char(3) 能存 ‘123’，也能存 ‘一二三’。 VARCAHR 同理。
 
 - TINYINT、SMALLINT、MEDIUMINT、INT 和 BIGINT 的长度指的是显示宽度
 
 其实和数据的大小无关，因为每种整型类型的可存储范围都是固定了的，指定长度是无法更改其存储范围的，只能改变 0 填充时整数的显示宽度。举例说明：
 
 ```
    CREATE TABLE `t` (
      `ui` int(20) unsigned zerofill DEFAULT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8
    insert into t values(123456789);
    -- select 结果为
    00000000000123456789
 ```
 
 - FLOAT、DOUBLE 和 DECIMAL 的长度指的是全部数位
 
 也包括小数点后面的小数个数，例如 **decimal(4,1) 指的是全部位数为 4**，小数点有且必须有 1 位，整数最长只能 3 位数。如果插入 1000（纯整数），则在高版本 mysql 中会报 *out of range* 错误，此时能插入的最大值是 **999.9**。
 
 - TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT 的长度
 
 tinytext/mediumtext/longtext 不能显示指定长度。
 
 在 utf8 编码格式下（mysql 5.7），text 指定的长度如果是从 0~21845，则其就为 text 类型；如果指定长度为 21846~5592405，则自动增长至 mediumtext 类型；如果指定长度为 5592405 到 4294967295 则自动增长至 longtext 类型。


# 选项和属性

## zerofill - 定义数据的显示宽度

```
    alter table tb_name add `new_column` tinyint(3) zerofill;
    insert into tb_name (`new_column`) values(1), (10);
    select new_cloumn from tb_namel;
    -- 结果为
    001
    010
```

zerofill 不影响数据的存储范围。宽度大的不影响，不会截取显示。

## 字段的最大长度

类型本身的限制和记录的总长度。不同字符集下也会影响。

not null 会影响字段长度，通常是占用 1 位。

## 列属性

 - 是否为空：`null/not null`
 
 null 和空字符串谁占空间？（null）
 
 - **默认值**
 
 当没有为字段设置值的时候自动启用，使用格式：`default VALUE`，默认值的设定需要固定值。
 
 如果一个字段没有默认值而且为空，则会插入失败。举例说明：
 
 ```
    create table tb_name (
    	a int(10) not null,
      	b int(10) not null
    );
    insert into tb_name (a) values (10);    -- 不会报错
    insert into tb_name (b) values (20);    -- 会报错："Field 'a' doesn't have a default value"
 ```
 
 上面的语句插入不成功的原因是第二句插入语句没有 a 同时 a 也没有默认值，第一句没有报错是因为顺序问题：
 
 > **当给字段赋值的时候如果不从首字段开始则前面不能省略，而是将需要赋值的字段名都要写出来然后写出字段对应的值。**
 
 - **主键**
 
 Primary Key。可以由唯一标识某条记录的字段或者是字段的集合组成（组合主键），不推荐组合主 键，因为比较麻烦。
 
 此外，虽然主键可以是真实实体的属性，但也不推荐，最好的是用一个与实体不相关的属性作为唯一标识。主键最好不与业务发生关系而只做标识用。
 
 主键不能相同，也不能为 null（默认），否则插入失败。
 
 主键可以为负，但是否为负取决于主键的类型。
 
 只能存在一个主键。常见的设计中，都存在字段，并且不是组合主键。
 
 ```
    -- 定义主键
    primary key (cloumn_a, cloumn_b)
    -- 删除主键
    alter table tb_names drop primary key;
 ```
 
 定义主键可以在字段类型后面马上定义，或者最后单独定义。
 
 其中组合主键只能最后单独定义主键的时候将多个字段都写进去。
 
 组合主键的含义是一个主键包含了多个字段而不是多个字段都是主键。
 
 - **自动增长**
 
 为每条记录提供一个唯一的标识，将某个字段的值自动增加 1，一般是从 1 开始增长，但也不一定。可以修改表选项来设置增长的初始值。 如从想要从 n 开始自增：
 
 ```
    alter table tb_name auto_increment N;
 ```
 
 这种情况，如果设置的值小于当前已自增的最大值，则会无视设置，继续以前的自增。
 
 **自动增长的条件：整型和有索引（通常是主键）**。为主键设置自动增长通常有 2 种情况，一是在现有主键基础上设置自动增长，二是删除现有主键之后，重新在新建主键同时指定主键自动增长。自动增长遇到 NULL 会自动 +1。
 
 **强制自增：** 具有 auto_increment 属性的字段，插入记录时无论该字段插入为 null 或是不插入值，都会强制自动增长。
 
 如果删除最后一列后，从删除处开始继续增长；如果从中间删除，则依然从当前最大值开始自增。同一张表中删除过后的字段序号相当于作废，与当前最大值比较，如果大于当前最大值则继续使用，反之完全作废。
 
## 设置表存储引擎和字符集

设置存储引擎可以使用等号也可以不适用，设置字符集可以使用 `charset` 也可以使用 `character set` 。为了统一都使用 `character set`。


# 约束/外键

## 定义

> 实体间的关系需要一些限制才能确保逻辑正常，约束就是这么一些限制。

**「外键」** 用于约束处于关系内的实体，`外键可以增加当前数据完整性，提高逻辑的可靠度。`比如在增删改查子表或父表时，父表或子表应该如何处理相关记录。

如果实体 1 的某个字段，指向／引用实体 2 的主键，则该实体 1 的该字段就是外键（Foreign Key）。

被指向的实体 (表) 称为主实体（主表，父表），负责指向的实体，称之为从实体（从表，子表）。

**举例说明：**

```
    -- TABLE_A 具有字段 `CLOUMN_A`
    -- TABLE_B 具有主键 `b_id`
    -- foreign key (外键字段) references 主表名 (关联字段)[主表记录删除时的动作][主表记录更新时的动作]
    CONSTRAINT `TABLE_A_ibfk_1` FOREIGN KEY (`CLOUMN_A`) REFERENCES `TABLE_B` (`b_id`) ON DELETE CASCADE
```

*外键表实体间的关系，在实体外定义。主键表实体内部字段的关系，在实体内定义*。

## 标志

外键字段在使用 `desc tb_name` 时会是 MUL，代表 **multiple**，多个关联的意思。

创建外键的时候 `ibfk_1` 是系统默认提供的标志符。

## 主表和从表

对子表的操作会收到主表的限制。比如，如果主表中没有记录，那么插入从表是不会成功的。

反过来没影响，即新增主表记录时不会收到从表记录的影响。

## 级联

 - **主表更新时从表行为**
 
 ```
    主表行为 => on update
    
    从表级联
    => cascade  => 跟随主表行为
    => set null => 从表不再指向任何主表【前提：主表中关联从表的那个字段属性不能设置成 not null，否则 set null 不会成功】
    => restrict => 拒绝，默认行为（因此执行 `show create table tb_name` 时不会显示）
 ```
 
 - **主表删除时从表行为**
    
 ```
    主表行为 => on delete
    从表级联 => 同主表更新
 ```
 
 - **修改级联操作属性**
    
 ```
    -- 1. 先删除外键
    alter table slave_tb drop foreign key foreign_key_cloumn;
    -- 2. 重建外键
    alter table slave_tb add foreign key (foreign_key_cloumn) references master_tb(m_tb_primary_key) on update/delete cascade/set null/restrict;
 ```
 
 ***注意:***
 
  - on update 和 on delete 可以同时出现，但允许的级联操作（cascade/set null/restrict）在每个 on update 或者 on delete 上只能出现一次。
  
  - 外键只对支持外键的存储引擎起作用。

## 外键是否必须

> **作为 MySQL 系统管理员（即站在数据库的专业角度）必须会使用外键，但作为 PHP 程序员（即站在应用的角度）尽量少用外键而是用其他的方案替代，比如想办法通过字段之间的互相引用建立一个对应关系，而不必在增加表的时候设置 foreign key**。


# 基本SQL语法

## 插入

### 一次性插入多条记录

```
    insert into table_name( field0, field1 ) values( xxx, xxx ), ( aaa, aaa ) , (ddd, ddd ) ;
```

可以不将所有字段都插入数据，如果要完成部分字段的插入，则必须存在字段列表;没有插入的字段将使用默认值

### set 语法在 insert 中的应用

如果只插入部分字段，可以使用 set 更新数据。
    
```
    insert into tb_name set field1=value1, field2=value2, ...;    --  和 insert 基本用法功能相同
```

### 插入数据时主键冲突？

因为默认有主键约束，所以这种情况不会插入成功。但是在使用 insert 语法时可以指定为更新主键值。

```
    insert into tb_name (field1, field2, ...)
    values (value1, value2, ...)
    on duplicate key
    update field1=new_value1, field2=new_value2, ...;
```

上述 on duplicate key update 的含义是如果主键冲突则更新字段值（insert 语法中的 update 后面没有 set）。此类语句执 行的流程为：

先判断是否插入成功，如果可以插入成功则不会执行更新操作；如果插入失败(主键冲突或是唯一索引冲突)，则执行更新操作。

 - **蠕虫复制**
 
 就是将 select 本表的结果插入到同表中。
 
 除了使用自定义数据外，还可以使用 select 语句查询到的结果作为插入数据的来源进行插入
 
 ```
    insert into tb_name (field1, field2, ...)
    select cloumn1, cloumn2, ...
    from ...;
 ```
 
 这么使用要求插入字段，也就是 select 的对象，的类型和数量，要和插入字段列表里面一样。
 
 - **插入数据时默认值的设置方式**
 
 在插入值的时候，可以直接在值列表中插入 null ，也可以直接将 default 关键字，或 `default(field)` 函数作为插入的值 列表中的一个
 
 建议插入 `default`，因为如果插入 `null` ，当字段本身默认值不为 null 的时候会出现数据不一致。插入 `default` 关键字的时候，设置过的 `default` 值将自动被插入。 **默认值可以保证数据的完整性**。
 
 - **replace**
 
 如果 insert 由于类似主键冲突，唯一索引等错误而插入失败时，使用 replace 替换 insert 可以将新插入的记录去替换， 而没有类似错误时功能同 insert 。但建议除非在十分确认的情况下才使用 replace ，因为如果失误，则原有数据会丢失。语法同 insert 。
 
 - **导入数据**
 
 ```
    load data infile '/path/to/file' into table tb_name
 ```
 
语法同 `select into outfile`，`load data infile` 专门倒入由 `select into outfile` 导出的文件，也有和 `select into outfile` 相同的配置命令。

**注意：** 导入时，涉及到数据的增加，此时需要考虑到数据是否冲突。通常，可以在导出时，将主键导出为 null ，利用自动 增长的特性，可以形成新的主键。

## 更新

### 更新数据的方式

 - replace
 - insert on duplicate key update
 - update：update 也支持 order by/where/limit
 - set
    
 ```
    set profiling=1;    -- 设置允许记录 sql 精确的执行时间
 ```
 
 - 多表更新
 
 使用 join。
 
 ```
    update table1
    join table2
    on table1.field1=table2.field2
    set field3=value1, field4=value2
    where field5=value3;
 ```
 
多表更新和多表删除的优点是可以一次性执行多步操作，缺点是复杂并容易忽略掉一些细节。

 - 关于 rename
 
 只有表存在 rename 操作，数据库不存在。
 
 rename 支持跨数据库重命名表。使用举例：
 
 ```
    -- 修改表名
    rename table
    old_tb_a to new_tb_a,
    old_tb_b to new_tb_b,
    ...;
    
    -- 交换两个表名
    rename table
    table1 to tmp,
    table2 to table1,
    tmp to table2;
 ```
 如果相对数据库进行重命名，可以参考的做法有：

 ```
    # 1. 直接修改文件夹名字 [不通用=>不推荐]
    # 2. 先将旧数据库导出，在新建一个数据库，再导入，导入的过程中就完成了重命名操作，最后删除旧数据库
    # 3. 将旧数据库的表移动到新数据库（rename)，然后删除旧数据库   
 ```
 
 - 表的基本修改操作
    
 ```
    -- alter table tb_name add|modify|drop|change column_name ...
    -- 新增列／字段
    alter table tb_name add new_column column_defination [after|brefore column];
    
    -- 修改字段定义
    alter table tb_name modify column_name new_defination;
    
    -- 删除字段
    alter table tb_name drop column_name;
    
    -- 重命名某列
    alter table tb_name change old_column new_column new_defination;
    
    -- 修改表选项
    alter table tb_name new_tb_options;
 ```

 - 修改字符编码
 
 ```
    -- 修改数据库字符编码
    alter database db_name character set names utf8;
    
    -- 修改表字符编码
    alter table tb_name character set utf8;
    
    -- 设置客户端编码
    set character_set_client = 'utf8';
 ```

## 删除

 - `if exists` 是否必要？
 
 如果不写 if exists，当表或数据库不存在时，则会报错删除失败。
 
 - 语句没错，但是仍然删除错误的原因可能是什么？
 
 有时候由于安全限制，可能操作系统不允许删除文件，则应该先在文件系统对文件操作的级别上删除 文件，然后再使用 drop 命令删除表。
 
 - 如何有条件地删除数据？

 - delete + limit
 
 ```
        delete from teble limit n;    -- 被删除的 n 条记录是 selete * from table; 中的前 n 条记录
 ```
 
 - delete + order by + limit
   
 ```
    delete from table order by field_name limit n;    -- 只有 order by 没有 limit 没有意义
 ```
 
 -  delete + join
   
 连接删除，同时删除多个表内的记录（不使用外键技术，因此存储引擎不会限制在 innodb）。
   
 需要先提供表名，再提供连接条件：
 
 ```
    delete from table1, table2
    using pub_field
    join table2 on field1=field2 where field3=value;
 ```
 
 此命令可以拆分成：
 
 ```
    delete from table1 where field3=value;
    delete from table2 where filed3=value;
 ```
 
 - truncate tb_name
   
 清空表，作用同 `delete from tb_name`，两者区别在 `truncate tb_name` 不会返回记录数，并且会重建自动增长的主键（从 0 开始），本质是先删除表再重建表，同时不会具有原来表的属性。
   
 而 `delete from tb_name` 会返回记录数，不会重建自动增长的主键，本质是逐行删除

## 查询

### show

```
    show create dabatabse db_name;    -- 查看数据库定义
    show create table tb_name;        -- 查看表定义
    show databases;                   -- 查看当前数据库系统中存在的所有数据库名
    show tables;                      -- 查看当前数据库的所有表名
    show character set;               -- 查看支持的字符集
    show collation like 'utf8%';      -- 查看支持的校对规则
    show variables like "char%";      -- 查看 MySQL 内置变量
    show full processlist;            -- 查看 MySQL 进程
    show profiles;                    -- 查看历史 SQL 语句的执行精确时间 Duration*1000 为毫秒数
```

### select

#### 多字段排序

先按照第一个字段排序，如果不能区分，则按照第二个字段排序，以此类推。

```
    -- asc => ascending 升序 默认
    -- desc => descending 降序
    select * from tb_name order by cloumn_a asc, cloumn_b desc;
```

排序关系受校对规则和字符编码影响。

#### limit

```
    -- 格式：limit OFFSET, ROW_COUNT
    -- OFFSET => 偏移量
    -- ROW_COUNT => 要截取的长度 不是指结束位置
    -- !!! OFFSET + ROW_COUNT 的索引位置从 0 开始而不是从 1 开始
    -- OFFSET 不是 MySQL 关键字而是具体的数字 => 内部索引值
    -- OFFSET 可以不设置 因为有默认值 0
    -- 若 ROW_COUNT 值超过索引最大值 则 MySQL 会自动截取
```

#### 去重

重复记录指的是字段值都相同的记录，而不是部分字段值相同的记录。

```
    -- 筛选 CLOUMN_NAME_A CLOUMN_NAME_B CLOUMN_NAME_C 都相等的值 只留一个
    select distinct CLOUMN_NAME_A, CLOUMN_NAME_B, CLOUMN_NAME_C from tb_name ;
```

`select distinct *` 和 `select * ／select` all效果是一样的。

#### 子查询

有些时候查询的条件需要先使用另一个查询语句获得，因为有时如果不先获得一个中间的查询结果，再借助该结果来查询需要的信息，则可能会使查询结果不准确，比如被遗漏。

比如，查询代课天数最多老师的姓名和性别信息，如果用 `select name, gender from teachers order by days limit 1`，则当有多个带个天数相同的老师的时候，也只能获得一 个结果，由于事先并不知道代课天数相同的老师有多少个，所以 limit 在这个在这里并不好限制。此时可以换个思路：可以先通过查询获得最大的代课天数，然后再通过将这个最大的代课天数作为下次查询的条件，这样只要符合条件的记录都会出现在结果中而不会有遗落的情况发生。

像上面这种情况，如果分条写则显得有点啰嗦，刚好 MySQL 可以不用变量的方式而可以直接将中间查询语句的查询结果作为值，来作为另一条语句的限制条件。

如果一个查询语句出现在了另一个查询语句中， 即父语句内部有子语句，称为子查询。使用子查询，可以将一个比较复杂的逻辑分步进行管理，降低业务逻辑的复杂 度，同时使查询一步到位。

```
    select name, gender from teachers where days = (select max(days) from teachers);    -- 这里的子语句需要使用括号包裹起来，否则 MySQL 不能解析
```

#### 子查询的分类

 - 根据 subquery 出现的位置
   - **where 型：** 详见返回形式为单一值，行和列的情况
   - **from 型：** 详见返回形式为表的情况
   - **exists 型：** 使用格式很简单 => exists(subquery)。返回的是布尔值，即如果子查询能够返回数据，则认为 exist 表达式返回真， 否则返回假。 举例说明:
    
 ```
    select * from tb_a where exists (select * from tb_b where tb_a.field_a = field_b);
 ```
 
 其中，field_b 是 tb_b 里面的字段，所以不用指明表名。
 
**exists 型查询的特点？**

类似于双重循环，普通查询只需要判断一次，符合即查询成功，不符合则查询失败。而 exists 查 询，每在父句（表1）中遍历一次获得每一个值，都需要在子句（表2）中遍历所有记录来检查有无符合条件的记录存在，即外部执行一次，内部执行多次，和双重循环有点类似。

**exists 与 in 的联系与区别？**

都能完成相同的事情，但思维方式不一样。

in 是先获得表 2 中所有所需的字段，然后再与表 1 中所需查询的一个或多个字段进行对比，判断这一个或多个字段是否在 in 检索出来的集合内。两者没有谁有谁劣的说法，各有各的优点。

如 exists 的优点是不用把所有结果都保存起来，数据量大时比较节省空间，而 in 的优点是将所有符合条件的结果事先存储起 来，这样检索的时候速度会快很多，一个是空间效率高，一个是时间效率高。存在即有理，使用何种方式取决于具体业务使用哪种更加方便。

- 根据 subquery 的返回值
  - **单一值**
  
  返回值是单一值的查询称为`标量子查询`。即获得一个值后，使用基本的关系运算符进行判断。
  
  - **列（一列，多列）**
  
  称为列子查询。通常是获得多个行的一列值。可以是一个范围，也可以是一个集合。举例如下:
  
```
    -- 返回一列
    select name from teachers;
    
    -- 返回多列
    select name, gender, days from teachers where name in (
      select name from teachers where class_name='php'
    );
```

  - **行**
  
  返回值为一行的称为`行子查询`。要返回一行可以不使用子查询获得：
  
  ```
    select * from tb_name limit 1;
  ```
  
  或者使用行子查询：
  
  ```
    select name, gender, class_name from teachers where (gender, class_name) = (
      select distinct gender, class_name
      from teachers
      where gender='male' and class_name = 'php'
    );
  ```
  
  这里要返回一行则需要使用括号先构建出一行，并且构建行的结构要与子查询语句获得的信息结构一致，然后再使用逻辑运算符连接子查询。
  
  - **表(多行，多列)**
  
称为表子查询。举例说明:

```
    select * from (
      select name, gender, class_name
      from teachers where days > 15
    ) as tmp where name like '李%';
```

如果表子查询用在 from 后面，则要求 from 后面跟着的是一个表而不是查询结果，所以需要将子句的查询结果用 as 起名使其成 为一个表，然后在被父句所检索。

表子查询的核心是将子句查询结果看作一个表。然后再从这个表中进行二次查询。这也导致了父查询使用字段名将由子句指定。


#### 子查询的操作符

集合操作符。常用 `in` 或 `not in` 表示。

 - `=any(集合)`
 等价于 in。等于集合中的任何一个即在集合中有则成立。
 
 - `!=any(集合)`
 表示不等于集合中的任何一个即可，也即只要与集合中的某些元素不相等则成立。
 
 - `!=all(集合)`
 等价于 not in。不等于集合中所有元素即不存在于当前集合中。
 
 - `some(集合)`
 和 any (集合) 在 MySQL 中没有区别。兼顾英语水平高的程序员的理解，无需多做解释。
 
**注意：** `all`，`any`，`some` 比 `in` 和 `not in` 要强大，可以使用除了 `=`, `!=` 之外的运算符，不过要复杂些，用得不多。

`!=any(集合)` 实际开发基本上不用而面试时却常考。


#### 联合查询

当需要查找同时符合多个条件的记录时，联合查询可以一步到位而不需要分几个步骤来执行，这样可以同时输出符合条件的 多条记录。

当获得数据的条件出现逻辑冲突或者很难在一个逻辑内表示，就可以拆分成多个逻辑，分别实现最后将结果合并到一 起。

将多条 select 语句的结构合并到一起，而多条 select 语句的语法并没有发生变化的操作，就是联合查询。
    
```
    (select c1, c2 from tb_name where c3=val1 order by c4 desc limit 3) union
    (select c1, c2 from tb_name where c3=val2 order by c5 asc limit 4)     -- 为了可读性和可维护性 子查询推荐使用 `()` 包裹起来
```

***注意:***

- 由 union 联合起来的多个 select 语句检索到的字段数要一致，数据类型上也应该严格一致。虽然 MySQL 内部会做类型转换处理，但在数据类型差异比较大的时候不一定能转换成功。比如：

```
    select 1 + 'a';    -- 结果为 1
```

但是检索的具体的字段名却可以不同。此外，检索结果中的列名称由第一条 select 语句的列名而定。

- 如果 union 查询的结果存在重复的记录，那么默认会自动消除重复。使用 union all 代替 union，可以将包括重复记录的符合条件的记录全部输出。

**使用 union 对所有结果进行统一排序的特点**

 - 子语句的 order by 只有在配合 limit 时才能生效，原因是 union 在做子语句时，会对没有 limit 的子句的 order by 优化。（忽略）
 
 - 只需在最后一个 select 语句后面增加排序规则，因为 union 的特点就是将多个 select 语句处理后对用户显示出只有一条 select 语句的结果。举例：
    
```
    (select c1, c2 from tb_name where c3 = val1) union
    (select c1, c2 from tb_name where c3 = val2 order by c4 limit 5)
```

这里，第一个子查询不需要使用 `order by`，而第二个如果需要就必须配合 `limit` 才有效。因为由 `union` 连接的多条 `select` 语句可以看作一条 `select` 语句对待

#### 连接查询

如果将一个表设计的比较复杂，则违背数据库设计的三范式原则，所以如果一个业务逻辑需要使用多个实体的数据时（一个 实体一张表），尽量不要将多个实体的数据设计在一张表内，将一个比较复杂的表分割成多个简单的表，这样可以减少依赖。

但仅仅是分割而不连接则业务所要使用的数据是不完整的，所以需要将多张表的记录连接起来一起使用。使用连接查询，可以想象成数据库用一些特定的方式（即连接查询的实现方法）去维护了一张实际上并未存在的大表，这张大表就是由要连接的多张实体表组成的。

**连接查询总体思路：** 将所有的数据按照某种条件连接起来，再进行筛选处理。

连接查询有时候也被称为连结查询。根据连接条件的不同，可以分为：

- 内连接（Inner）
  - **交叉连接：** `cross join`，也称为或笛卡尔积连接。
    在 MySQL 中，`cross join` 和 `inner join` 相同，但在数据库的定义上，交叉连接就是笛卡儿积，是没有查询条件的 `inner join`。
  - **内连接：** `inner join`，MySQL 默认，等价于 join。
    真实存在的数据内部的连接。要求连接的多个数据都必须同时存在才能进行连接。
    
  使用格式：`select ... from left_tb inner join right_tb on` 连接条件。
  
  **左表和右表的区分：** 在 join 左边的就是左表，在 join 右边的就是右表，属于逻辑上的概念，谁左谁右并不影响。
  
  **省略连接条件的内连接：**
  
  内连接可以省略连接条件，这意味着左表的每一条记录都要与右表的每一条记录进行一次连接，假设左表有 M 条记录，右表有 N 条记录，则将有 M * N 个连接。这种情况下，通常使用 `cross join` 代替 `inner join` 来避免获得过多记录。
  
  当需要找出所有的可能性的时候多使用这种查询方式。

  **有条件的内连接：**
  
  会在连接时过滤非法的连接。指明条件的关键词有 `on, where, using`。虽然它们的连接结果是一样的，但是逻辑含义不一样。它们的区别和使用场景是：
  
```
    on => 连接过程中对数据进行判断时使用的条件，在通用条件下使用。
    
    where => 交叉连接成功后在新表数据中的过滤条件，在数据过滤时使用。
    
    using => 负责连接的两个实体之间的字段名一致。是一个快捷语法，需要在同名字段的环境下使用。使用格式是：using (两个实体表的同名字段)。using 会去掉重复字段并将 using()括号中的字段放在列前。 
    
    -- 举例说明
    select id, name, days from teachers inner join teachers using(id);
    
    -- 说明
    这里虽然可以通过 id 字段查询到记录，但是记录是不准确的。
    最好不要用 id 作为两张实体表的同名字段，也不能写成类似 `using(a_id=b_id)`，因为 using() 里面只能是一个字段。而不是表达式
```

  - 逗号：,
  
  可以不使用 where，而直接使用多表查询，来实现笛卡儿积，比如：
  
  ```
    select tb_a.c1, tb_a.c2, tb_b.c3
    from tb_a, tb_b;
    -- 结果
    c1  | c2  | c3
    ... | ... | ...
  ```
  
  此时的查询结果仍然相当于一张表查出来的。
  
  **内连接的查询条件和外连接通用，不过外连接不使用 where 作为连接条件。**
  
  - 外连接（Outer）
  
  如果负责连接的一个或多个数据不真实存在，比如表 1 外连接表 2 时，当表 1 中的某条记录在表 2 中没有与之匹配的记录时，则称之为外连接。
  
  此时，虽然表 2 没有与之配对的记录，但表 1 中的那条记录是确实是存在的，这时就只能用外连接。
  
  使用格式举例：
  
  ```
    -- select ... from table1left left outer join table1right on 连接条件
    
    -- eg
    select teachers.name, class.begin, class.days
    from teachers left outer join class on teachers.id=class.t_id;
  ```
    
  outer 关键字可以省略，因为内连接没有左右之分。
  
  此外，左右连接本质都是一样的，看站在什么角度去理解。左表、右表之分是与 join 的位置关系有关的，跟 left 、right 无关。
  
  - 左外连接：`left join`
  
  在连接时，如果出现左边表数据连接不到右边表的情况，则左边表的数据在最终结果中被保留，右边表内数据用虚拟记录代替（NULL）。
  
  左连接是开发中使用最多的连接方式，因为任何一个数据都不能保证在连接之内都能找到对应的数据配对。
  
  - 右外连接：`right join`
  
  在连接时，如果出现右边表数据连接不到左边表的情况，则右边表的数据将被保留。
  
  - 全外连接：`full join`，MySQL 暂不支持，实质是左连接结果 union 右连接结果，比如可以模拟实现全连接：
  
  ```
    (select * from a left join b on a.id = b.aid) union
    (select * from a right join b on a.id = b.aid);
  ```
  
  - 自然连接：`Natural`

既包含自然内连接又包含自然外连接。通过 MySQL 自己的判断完成连接过程，而无须指定连接条件，MySQL 会自动使用 几张表中相同的字段作为连接条件。使用格式为：`select * from a natural right join b`。

被自然连接选中的同名字段在输出的结果中靠前，同时，使用什么自然连接（左／右），就先将什么表的字段先显示。

自然连接也分为自然内连接和自然外连接，同时自然外连接又分为自然左外连接和自然右外连接。默认为自然内连接（不需要 inner 修饰）。

自然连接查询和使用 using 条件查询结果一致，即以下一些语句是等价的：

```
    select * from one natural join two;
    
    -- 等价于
    select * from one inner join two using(public_field);
    select * from one natural left/right join two;
    
    -- 等价于
    select * from one left/right join two using(public_field);
```

  - 连接条件
    - on：连接条件。
    - using：使用同名字段快速连接
    - where：筛选条件，**注意：不能使用没有条件的外连接**

  - **内连接和外连接的区别**
  
  内连接只保留了连接上了的记录，外连接保存了连接上了的和没有连接上的记录;如果保存的是左表的记录则就是 左外连接，如果保存的是右表的记录就是右外连接。
  
  - 连接查询的步骤
  
  所有连接的过程都是相同的，也类似与多重循环：
    
  ```
    -- 1. 连接左表第 1 条记录和右表的第 1 条记录
    
    -- 2. 判断条件(on)，符合则将连接结果保留，否则不保留
    
    -- 3. 继续连接，左表第 1 条记录依次和右表第 2, 3, ... 条记录，然后判断
    
    -- 4. 当左表第 1 条记录与右表所有记录连接完毕后，开始重新连接左表第 2 条记录与右表第 1, 2, 3, ... 条记录，以此类推
  ```
  
#### join 和 union 的区别

union 是记录之间的组合，join 是字段之间的组合。

### 将检索结果导出到文件

```
    select * into outfile '/path/to/file'
    from tb_name
```

`select into outfile` 不能创建文件夹但能，也只能创建新文件。

此外，如果创建的文件名称和指定路径中的文件夹/文件名称 相同则不会创建成功，这样可以保护现有文件不被重写。

`into outfile` 适用于 `select` 所有查询语句。

### 生成文件格式

默认采用行来区分记录，制表符来区分字段。但为了满足某种特殊的需求，会采用不同的分割方式，如 cvs。

MySQL 支持在导出数据时候设置记录与字段的分割符：

 - fields => 设置字段选项
 - lines => 设置行选项
 - enclosed => 设置字段分隔符
 - escaped => 设置字段包裹符
 - terminated => 设置字段终止符，后接 by ‘xx’ 即可
 - starting => 设置行开头，后接 by ‘xx’ 即可

```
    select * into outfile '/path/to/file'
    fields terminated by ','
    lines terminated by '\n'
    starting by 'start:'
    from tb_name where conditions;
```

- 导出二进制数据

```
    select into dumpfile ...
```

此时将会把数据库内的数据以二进制形式保存到服务器文件。

导出的文件中只有一个文件结束符 EOF 而没有其他的格式，普通文件还是能被解码识别，只是没有格式可读性很不好，本身为二进制文件的 文件就会以”乱码”形式表现出来。

不过，这种情况不常用，因为数据库大都不直接保存二进制文件而是保持二进制文件的保存路径即可。


# 视图

## 为什么需要视图

出于权限安全和隐私的保护需要。有时候保存在某张表内的记录，不希望所有字段都能被所有人查看和操作，这时可以将原表中的一些允许被查看和修改的字段通过 select 选择出来后重新形成一张虚拟表，以替代对原表各种操作从而保护原表中不希望被查看的数据。

此外，视图还可用于替换复杂的 select 语句，如很长的连接查询语句，从而隐藏复杂的业务逻辑，通过视图完成的逻辑通常都是很简单的逻辑。

## 什么是视图

对`视图（本身是张虚拟表）`主要进行的操作是查询，不主要用于更新和删除，因为对视图的操作最终会影响到原表，而原表那些被隐藏起来的字段通过操作视图是不能够更改的，所以就算是更新和删除了视图，原表中的数据也是不完整的。

视图主要提供给外部人员查看的，而非更改的，相当于提供了一个外部接口。视图本身不保存数据，只是通过 select 语句查询相关数据，**视图本质只保存了一条 SQL 语句而已**，利用视图对原表进行查询。每对视图进行一次操作 都会执行保存的那句 SQL 语句。

## 如何使用视图

```
    create view view_name as select statement;
```

创建视图后就相当于在数据库中形成了一张虚拟表。

如果需要外部接口，一个数据库里面有多个应用，则针对每一个应用采取不同的视图接口。


## 视图管理
    
```
    -- 删除视图
    drop view if exists view_name;
    
    -- 修改视图
    alter view view_name as select statement;
    
    -- 修改视图结构
    alter view view_name(new_field_1, new_field_2, ...) as select statement;    -- 修改视图内所使用的字段的名称
```

*说明*

对视图的修改即是对原表进行修改，即时生效。删除视图时，不会影响原表。

## 视图的执行过程和执行算法

视图的执行算法指一个视图在什么时候执行，依据哪些方式执行。

通常有 3 种执行算法。

- **undefined**
当创建视图时， MySQL 默认会使用一种 undefined 的处理算法，即 MySQL 会自动在合并和临时表中进行选择。

- **merge**
合并的执行方式，每当执行的时候，先将视图的 SQL 语句与外部查询视图的 SQL 语句混合在一起，最终执行。

- **temptable**
临时表模式，每当查询的时候，将视图所使用的 select 语句生成一个结果的临时表（(select … as temp），再在当前的临时表内进行查询。


# 事务

## 为什么需要事务和什么是事务

有时候，业务需要多条 SQL 语句的同时执行成功才能算完成。一组具有相同业务逻辑的 SQL 语句单元，组内所有 SQL 语句完成一个业务。如果整组成功，代表全部 SQL 都实现了，则业务成功；如果其中任何一个 SQL 语句执行失败， 则意味着整个操作都失败，数据库将回到操作之间的状态，则业务失败。

以上特性就是事务。事务没有成功之前，别的用户（进程，会话）是不能看到操作内的数据修改的。

## SQL 执行的阶段

 - 1、执行阶段
 - 2、将执行结果提交到数据库
 
 其中事务日志就是用于保存执行阶段的结果。默认的提交方式是自动提交。
 
## MySQL 如何处理事务

利用 Innodb 存储引擎的事务日志功能。事务机制只在 innodb 和 bdb 存储引擎下有效。

在一组操作之间设置一个记号（备份点），如果失败至少一个则从备份点处还原，如果都成功则提交。

事务处理的具体操作步骤为：

- 关闭自动提交功能

通过设置系统变量 autocommit 来配置自动提交选项：

```
    set autocommit=0;
    show variables like 'autocommit';
```

- 执行完所有 SQL 语句

如此设置后，如果再次执行更新语句，当前会话的修改会暂时生效，但其他会话查看数据库时还是没有任何改变，因为结果没有被提交。

- 判断是否都成功

如果成功则将结果提交（commit）。

如果失败，如出现语法错误，逻辑错误，服务器错误等，则回到开始位置`（rollback）`。

上述操作不是很方便，因为要使用事务逻辑的情况不是最多。所以最常见的事务指令为：`start transaction/begin`（不推荐使用 begin 因为它在 SQL 语法中有特殊的含义）。

使用该指令，如果事务结束了（无论业务成功与否），会根据处理结果自动提交或者是回到 start 之前的状态。

事务结束之前使用 `commit` 或 `rollback` 选择事务的处理结果。

## 事务的特点

### 原子性／A
Atomicity。不可再分，虽然在语法上可以将事务分成多个 SQL 语句，但在功能上不可以分割。

### 一致性／C
Consistency。事务开始之后和结束之前只受一个会话影响，事务操作的数据在事务结束之前会被锁定，其间其他会话不能修改。

### 隔离性／I
Isolation。多个事务之间不会相互影响。隔离级别／严重程度。

### 持久性／D
Durability。事务一旦提交会对数据产生永久影响。


## 锁的概念

对事务进行操作的时候会将整个表都上锁，所以其他会话不能操作处于事务之中的数据。


# 触发器

## 什么是触发器 ?

> 类似于 Javascript 中的监听事件（事件驱动编程），在 MySQL 中，可以监听数据进行操作。

在当前表上，设置一个对每行数据的一个监听器，监听相关事件，每当事件发生时，会执行一段 SQL 功能代码。

*触发器相当于一个程序。触发，特定时间的发生即触发。*

事件有：`insert`/`delete`/`update`。

数据，就是触发该事件的记录。

事件的时机：执行前和执行后。由时机和事件一起形成了 6 种事件：

```
    before insert
    before delete
    before update
    after insert
    after delete
    after update
```

监听位置有：for each row。

## 创建触发器

格式为：`create trigger 触发器名 事件 执行性代码`

事件可以规定为：在哪个表的什么时机上的什么动作。可执行性代码即 SQL 语句。比如：

```
    create trigger trigger_name
    after update
    on tb1 for each row update tb2 set field=value|statement;
```

## 管理触发器

```
    -- 删除触发器
    drop trigger trigger_name
    
    -- 查看触发器
    show create trigger trigger_name
```

### 获得触发器内的数据

在MySQL中用 `old` 和 `new`表示执行前和执行后的数据。

 - **old：** 监听事件所在表上的数据，在事件发生之前的数据，旧的数据。
 - **new：** 监听表上，时间发生之后新处理完毕的数据。

### 注意事项

 - 触发器不能同名。
 - 目前 MySQL 只支持一类事件设置一个触发器。
 - insert 事件不能使用 old，delete 事件不能使用 new。
 - 如果一个触发程序由多条 SQL 语句组成，应该将多条 SQL 语句用 begin 和 end 包裹起来。
   
   由于触发器中的 SQL 语句也是用分号 ; 作为语句结束符，这和命令行中采取的 ; 作为语句结束符就相矛盾了，就会导致 MySQL 客户端在执行创建触发器的相关 SQL 语句时会以触发器中 SQL 语句的 ; 作为结束符，而不是整个创建触发器的语句结束符。
   
   所以需要使用 delimiter 设置其他符号（通常为 $$ 符号）作为创建触发器时命令行中的语句结束符，创建触发器完成后恢复原来的结束符。举例说明：
   
```
    drop trigger trigger_name;
    delimiter $$
    create trigger trigger_name
    after insert on table_name
    for each row
    begin
    update tb_1 set field1=value1/expression;
    update tb_2 set field2=value2/expression;
    end
    $$
    delimiter ;
```


# 数据管理

## 备份

### mysiam 表

可以直接将 table_name.frm，table_name.myd，table_name. myi 三个文件拷贝一份出来。

需要的时候，直接移动到相应的数据库目录内即可。注意，此方法如果去处理 innodb 表结构的文件，则只能用 show tables 看到表，但不能使用，因为数据和索引并没有被复制过来而只有数据的结构。

### innodb 表

- 通用备份方案
将建表结构与插入数据的 SQL语句生成并保存即可。还原只需重新执行 SQL 语句即可。
 
- 使用 MySQL 工具集
即 mysqldump，其备份的其实也是 SQL 语句。

它不是 SQL 语法的一部分所以不需要再 MySQL 客户端执行，可以直接单独执行。

配置好环境变量后可以在命令行下直接执行：

```
    # 备份整个数据库内的表
    mysqldump -u root -p db_name > /path/to/save_file
    
    # 备份多张表
    mysqldump -u root -p db_name table1 table2 table3 ... > /path/to/save_file
```

### 还原

重新执行备份好的 SQL 语句。

可以在 MySQL 客户端直接执行，也可以一次性将一个文件里面的 SQL 语句全部执行，使用命令为：**source 备份文件路径**。

不过事先要创建一个新数据库作为备份还原的目标，否则会提示 ‘no database selected’。

### 导出和备份的差别

导出的是数据，而备份的不仅是数据还有结构。


## 误操作恢复

> 1、数据无价。
>  
> 2、请使用 where，你要知道你在干嘛。

常见的数据库误操作情况有，没带条件地更新／修改了某张表所有记录的一个字段。比如，批量更新了一张表的所有图片路径；或者，删除了不该删除的记录。

这时候不急是假的，因为用数据是无价的，但是紧急之下，先让自己冷静，参考下面几种 MySQL 提供的可恢复手段。

### mysqlbinlog

> 同大多数关系型数据库一样，日志文件是 MySQL 数据库的重要组成部分。MySQL有几种不同的日志文件，通常包括错误日志文件，二进制日志，通用日志，慢查询日志，等等。这些日志可以帮助我们定位 mysqld 内部发生的事件，数据库性能故障，记录数据的变更历史，用户恢复数据库等等。
>  
> MySQL binlog 日志记录了 MySQL 数据库从启用日志以来所有对当前数据库的变更。binlog 日志属于二进制文件，我们可以从 binlog 提取出来生成可阅读的 SQL 语句来重建当前数据库以及根据需要实现时点恢复或不完全恢复。

`mysqlbinlog` 基本的命令参数包括起止时间，举例如下：

```
    mysqlbinlog --no-defaults --start-datetime='2016-12-14 09:00:00' --stop-datetime='2016-12-15 17:00:00'  ~/mysql-bin.000027 > ~/mysql_restore.sql
```

然后从生成的 mysql_restore.sql 搜索你刚才误操作的 sql 语句，然后提取出这之前的 sql 中包含你目标表的 sql 操作，删除误操作影响到的错误记录，然后重新执行这些 sql，达到恢复数据的目的

## 配置与排错

### 查看 mysql 加载配置的文件路径

```
    mysql --help | grep my.cnf
    
    # eg: order of preference
    my.cnf,
    $MYSQL_TCP_PORT,
    /etc/my.cnf
    /etc/mysql/my.cnf
    /usr/local/mysql/etc/my.cnf
    ~/.my.cnf
```

# MySQL 关键字

## 是否
    
```
    where <field> is null
    where <field> is not null
```

# PHP 操作 MySQL

## 准备工作

要操作 MySQL ，PHP 必须要充当为 MySQL 的客户端(客户端命令行窗口)，因为 MySQL 是 C/S 架构 的。PHP 是MySQL 扩展库函数来操作 MySQL的。在操作之前，有以下几点前提需要注意：

- MySQL 服务器已经开放接口／API

这是为了允许其他语言操作，在安装 MySQL 时已经设置好了，如果是分步安装的 MySQL，则需要安装服务端、客户端、和开发端（接口）。

在 Windows 上安装 MySQL 时，动态连接文件 libmysql.dll 就是其他程序（非 MySQL 自带客户端）可以操作 MySQL 必先找到的文件 ，如果只有 MySQL 客户端可以操作 MySQL 而其他语言却不能连接，则可以将 libmysql.dll 复制到 c:/windows 文件夹(一个公共位置)。

- 相应的客户端

客户端利用 MySQL 服务器开放的接口，完成编程，从而操作 MySQL 数据库。

对 PHP 而言，就是增加 一个模拟客户端的功能。

- 载入相应功能

即载入 PHP 的相关扩展。

通过设置 php.ini 文件，Windows 需取消 `;extention=php_mysqli.dll `前的注释。同时还需要设置扩展文件所在的目录：`extension_dir = "c:/wamp/php/ext"`。

载入成功后使用 `phpinfo()` 函数可以看见 MySQL 的相关区域。或者测试 MySQL 扩展库中提供的函数是否有效来确认。比 如，`var_dump( function_exists( 'mysqli_connect' ) )`;，若返回 ture 则代表载入成功。

## PHP 操作 MySQL 的几种方式

无论是哪种方式，整个过程都分为 2 步：发出操作命令和接受执行结果。

- mysql
操作 MySQL 通用性最强的操作手段。不过在较新版本的 PHP 当中已经被弃用。

- mysqli：mysql 的升级版本，智能操作 MySQL 数据库。

- pdo

数据操作对象，提供了面向对象的操作方法，是数据库抽象层几乎可以操作所有流行的数据库软件

## PHP 操作 MySQL 的流程

- 连接和认证：详见手册
- 选择数据库，设置编码

此步也可以不进行，因为编码的设置只是为了在客户端显示而不能改变数据保存的格式。单独选择数据库(use database_name)只是一种习惯性操作而不是必须操作。在选择表的同时也可以在其前面指定数据库。

- 执行 sql：详见手册
- 处理结果

处理成功的结果会有两种：**布尔值**和**结果集**。

布尔值可以直接使用，但是结果集就需要处理，通常都是用 fetch 方法从结果集中逐条提取纪录。最常见的处理方式如下：

```
    # fetch 函数内置结果集的纪录指针 可以在循环中移动
    while ($row = mysqli_fetch_assoc($res)) {
      $rows[] = $row;
    }
    
    # fetch_assoc => 返回值为下标为字段名（字符串）的关联数组
    # fetch_row   => 返回值为下标为数值索引的的索引数组
    # fetch_array => 返回值为混合数组
```

- 关闭连接 释放结果集（如果有）：详见手册。

# SQL编程

SQL 也算是一个编程语言，既然是编程语言就可以用于编程。如 trigger 其实就是 SQL 编程的简单应 用之一。SQL 编程就是将多条 SQL 语句组合到一起完成相应的业务逻辑。

SQL 也有通用编程语言的基本元素。

## 函数

分为`内置函数`和`用户自定义函数`。具体查询手册。

## 数值函数

- rand()

```
    -- 获得 0～1 之间的随机浮点数
    select rand();
    
    -- 获得 5 ～ 10 之间的随机数
    select 5+rand()*5;
    -- ...
```

- floor()

```
    select floor(rand()*5+5);    -- 取整
```

## 时间和日期函数

- **now()：** 获得当前时间。
- **unix_timestamp()：** 获得时间戳，时间戳本身无意义但是可以将时间戳转换成有意义的时间格式
- **from_unixtime()：** 从格林威治时间开始的秒数。如：select from_unixtime(unix_timestamp()); 会显示当前时间。

## 字符串函数

- **concat()：** 字符串连接
- **length()：** 统计字符串长度(单位是字节数)
- **char_length()：** 统 计字符数(字符数取决于编码方案)
- **substring： ** 截取字符串。使用格式为：源字符串，开始位置，截取长度(字符数)) ，其中开始位置指的是如果为正数则为从左边数，如果为负数则表示从右边开始数。如：select substring('Hello world', 2, 2);
- **left 和 right：** 分别代表从左边截取和从右边截取字符串。
- **lpad('将 被补足的字符串', 补足后的长度, '补的字符串')：** 用字符串补足字符串，如：select lpad( 'dddd', 10, '**' ) ;
- **md5(‘要加密的字符串’) ：** 对字符串进行 md5 加密。
- **password(‘要加密的字符串’) ：** 加密字符串，相当于两次的 sh1() 加密
- **sh1(‘要加密的字符串’)：** sh1 加密。

## 变量

字段名，系统默认变量，如 variables。

用户自定义变量：`set 变量名=变量值`。为了区分系统变量 和自定义变量，需要则自定义变量之前加 @ 标志，否则会报错。自定义变量可以通过 select 语句查看：`select @variable_name`; 或者在表达式中使用。

使用 select into 定义变量：`select field_list` 表达式 `into variable_list`;。使用举例：`select field_name from table_name where condition into @variable`;

注意，`select into @variable` 要求只返回一行，如果返回多行，则会报语法错误，或者返回的是最后一行。

set 定义变量时可以将表达式赋给一个变量：`set @variable = (select count(*) from table_name)` ;

### 变量的作用域和有效期

**作用域：** 用户定义的函数，是全局的（函数内可用），在函数内部定义的变量是局 部的

**有效期：** 一次会话期间（连接结束之前）都有效。以 @ 命名的变量相当于全局变量，函数内外都能使用。

### 变量的数据类型

即字段的数据类型。

数据类型 => 表结构；数据 => 记录。

### 流程控制

- 分支

```
    delimiter $$
    create function fun() returns varchar(20) begin
    if hour( now() ) >= 18 then
    return 'Late.'; else
    return 'Early.'; end if;
    end
    $$
    delimiter ;
```

- 循环

```
    -- 1-10 的和 
    drop function if exists sum1 ; delimiter $$
    create function sum1() returns int begin
    set@i =1;
    set @sum = 0;
    while @i<=10 do
    set @sum = @sum + @i;
    set @i = @i + 1; end while;
    return @sum; end
    $$ delimiter ;
```

**循环的提前终止：** SQL 语法中没有 break 但有 leave（跳出当前标签指定的循环）；没有 continue 有 iterate（跳出当前循环的本次循环）。举例说明：

```
    drop function if exists sum1 ; delimiter $$
    create function sum1() returns int begin
    set@i =0;
    set @sum = 0; sum:while @i<10 do
    set @i = @i + 1; if @i = 5 then
    leave sum; --iterate sum;
    end if;
    set @sum = @sum + @i; end while sum;
    return @sum;
    end
    $$
    delimiter ;
```

**注意：** leave 和 iterate 不像 break 和 continue ，不是根据它们所在的位置来决定终止哪个循环，而是由循环的标签来决定的（类似于 Javascript）。

循环的标签即给循环起的名字，即上面例子中的 sum。

- 其他流程控制

此外， SQL 语法中也有其他流程控制语法，如 switch，case 等。具体查阅手册。

## 运算符

在 SQL 语法中，`=` 是关系运算符，即判断左右两端值是否相等而不是赋值运算符，赋值运算符是 `:=` ，如 `select ... @user := '姓名'`。其他运算符同 PHP 。

## 注释

行注释：`#` 和 `--` 。块注释：`/* ... */`。


## 用户自定义函数

也称存储函数，即存储在数据库里面的函数。

用户自定义函数是最能体现编程的情况，主要是使用 SQL 语法去定义一个函数的功能。自定义函数也包含：函数／参数列表／函数体／返回值。

```
    drop function if exists sayHello ;
    
    delimiter $$
    -- 创建函数
    create function sayHello( username varchar(20) ) returns varchar(20)
    begin
    -- 函数体
    return concat('Hello', username) ;
    end
    $$
    delimiter ;
    
    -- 调用自定义函数
    select sayHello(', caoxl')     -- Hello, caoxl
```

**注意：** 用户自定义函数和数据库是绑定的，如果在调用自定义函数的时候没有指定具体的数据库，那么会报错找不到函数。

函数也可以用 `db_name.func_name` 调用。一个函数可以有多个参数，使用逗号分隔。

函数体内的变量名前没有 @ 的话，代表的是局部变量，但是局部变量有专门的声明方式：`declare variable`;。 比如：`declare i int default 0`;

## 练习

- 判断变量是否为 null

```
    if var is null then ...
    if isnull(var) then ...
```

- 模拟测试数据

```
    drop function if exists v_name;
    delimiter $$
    create function v_name() returns char(2) begin
    declare first_name char(16) default '赵钱孙李周吴郑王冯陈蒋李敏韩沈';
    declare last_name char(10) default '甲乙丙丁戊己庚辛壬癸';
    declare full_name char(2);
    set full_name = concat( substring(first_name, floor(rand()*16+1), 1 ), substring( last_name, floor(rand()*10+1),1 )) ;
    return full_name; end
    $$
    delimiter ;
```

# FAQ

## mysql 5.7+ timestamp 零值报错？

```
    vim /etc/my.cnf    # 配置文件路径因环境不同而不同
    # 找到 `[mysqld]` 一定要在这个字段内配置才能读取到
    [mysqld]
    sql_mode=NO_ENGINE_SUBSTITUTION
    service mysqld restart
```

## 为什么 MySQL 的字符集设置不能写成 utf-8 ?

因为在 MySQL 里面 - 有特殊含义，表示的是字符集下面的校对集。

# 其他

## 表前缀

有时候几个项目需要使用同一个数据库，表前缀用于在同一个数据库中查询逻辑表名相同的表。如 info_student，exam_student。

前缀后缀只是逻辑上的存在，MySQL 统一认为都是标识符。

## 特殊符号的含义

- `%`：MySQL 通配符，表示任何字符的任意组合
- `*`：表示所有字段
- `+0` ：表示将检索结果用整数形式输出

```
    select column_name+0 from tb_name;
    -- raw `column_name`: 2016-10-07 17:18:00
    -- res: 20161007171800
```

- `\G ：` 当数据量比较多，结构比较混乱的时候，语句末尾用 \G 代替 ; 可以使输出更易
- `\g：` 原样输出，同 ;
- `\c ：` 取消操作，避免执行 SQL 时 MySQL 报错
- `\q：` 退出 MySQL 客户端

## 表的别名和列的别名

有时候表名过长会无谓地增加 SQL 语句的长度。使用别名可以减少 SQL 语句的总长度，这样，无论从功能上还是从代码的 可读性上看上去都更加友好。所以如果表名过长，则需要一个简洁、清晰的别名。

列别名即字段的别名。往往多张实体表中的字段名都相同，如 id，这就有可能会造成理解上的困难 – 到底是谁的 id ?为了 解决这个情况，增强代码的可读性，就需要给列起别名。

表的别名和列的别名起名方式一样的，用 as 关键字就可以了：`table_name as t_n，field_name as t_n.f_n`。

## 标志符之间的 `.`

没必要都写上。看字段是否有冲突。如果有冲突的话则用点进一步指明字段的上级表，还冲突的话就继续用点指明上级数 据库名。如果不冲突就只写字段就够了。`database.table.field`

## MySQL 函数

- sum(field)：求字段内值的和
- group by(filed)：按 field 组排序
- max(field)：获得 field 字段内最大值

# 经验

> 学习程序设计，基本概念／基本思想是重点，语法是次要的。

# 参考

 - [MyISAM versus InnoDB](https://stackoverflow.com/questions/20148/myisam-versus-innodb)
 - [MySQL 二进制日志(Binary Log)](http://blog.csdn.net/leshami/article/details/39801867)



