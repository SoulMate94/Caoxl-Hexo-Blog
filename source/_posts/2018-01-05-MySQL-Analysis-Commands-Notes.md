---
title: MySQL 分析命令总结
date: 2018-01-05 18:32:41
categories: MySQL
tags: [MySQL, Database]
---


# MySQL 分析命令总结
站在 MySQL Parser 的角度，describe, desc, explain 是完全等价的（synonym）。

只是通常情况，DESCRIBE/DESC 被用分析表结构，而 EXPLIAN 被用于分析 SQL 查询执行计划（how MySQL would execute a query）而已。

其中，`DESCRIBE` 又是 `SHOW COLUMNS`/`SHOW FIELDS` 的别名。

为了简单，下面涉及到上面 3 个命令的都在 MySQL 5.6 下以 DESC 进行示例。

<!--more-->

# 获取表结构信息

获取表结构的语法为：

- desc

```
    {EXPLAIN | DESCRIBE | DESC} tbl_name [col_name | wild]
```

其中 `col_name` 代表列名，`wild` 代表通配符。

- show

```
    SHOW [FULL] {COLUMNS | FIELDS} {FROM | IN} tbl_name
    [{FROM | IN} db_name]
    [LIKE 'pattern' | WHERE expr]
```

**举例说明：**

## 获取完整表结构

```
    DESC $tb_name;
    SHOW COLUMNS FROM $tb_name FROM $db_name;
    SHOW FIELDS FROM $tb_name FROM $db_name;
    SHOW FULL FIELDS FROM $tb_name FROM $db_name;
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE 1;
    SHOW FIELDS FROM $tb_name FROM $db_name LIKE '%';
    SHOW CREATE TABLE $db_name.$tb_name;
```

## 获取某个字段结构

```
    DESC $tb_name $col_name;
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE `Field` = $col_name;
    SHOW FULL FIELDS FROM $tb_name IN $db_name WHERE `Field` = $col_name;
```

- 获取主键字段的结构
    
```
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE `Key` = 'PRI';
```

## 获取若干字段结构

可以使用通配符 `%` 和 `_` 。比如：

- 获取所有以 `a` 开头的字段结构

```
    DESC $tb_name 'a%';
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE `Field` LIKE 'a%';
    SHOW FIELDS FROM $tb_name FROM $db_name LIKE 'a%';
```

- 获取包含 `a` 的字段结构

```
    DESC $tb_name '%a%'
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE `Field` LIKE '%a%';
    SHOW FIELDS FROM $tb_name FROM $db_name LIKE '%a%';
```

- 获取包含 `a`，且字段名只有 `3` 个字符的字段结构

```
    DESC $tb_name 'a__';
    DESC $tb_name '_a_';
    DESC $tb_name 'a__'
```

- 获取属性值唯一的字段结构

```
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE `Key` = 'UNI';
```

- 获取类型为 `int(11)` 的字段结构

```
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE `Type` = 'int(11)';
```

- 获取为可以为 NULL 的字段结构

```
    SHOW FIELDS FROM $tb_name FROM $db_name WHERE `Null` = 'YES';
```


# 分析查询计划

分析 SQL 查询计划的语法为：

```
    {EXPLAIN | DESCRIBE | DESC}
    	[explain_type]
        explainable_stmt
        
    explain_type: {
    	EXTENDED
    	| PARTITIONS
    	| FORMAT = format_name
    }
    
    format_name: {
    	TRADITIONAL
    	| JSON
    }
    
    explainable_stmt: {
    	SELECT statement
    	| DELETE statement
    	| INSERT statement
    	| REPLACE statement
    	| UPDATE statement
    }
```

其中，`explain_type` 可以不指定，但同时只能选择一次。`format_name` 默认为 `TRADITIONAL`。


**举例说明：**

## 分析 SELECT

### 不指定 explain_type

```
    DESC SELECT * FROM $tb_name;
    --> 输出示例：
    +----+-------------+-----------+------+---------------+------+---------+------+------+-------+
    | id | select_type | table     | type | possible_keys | key  | key_len | ref  | rows | Extra |
    +----+-------------+-----------+------+---------------+------+---------+------+------+-------+
    | 1  | SIMPLE      | $tb_name  | ALL  | NULL          | NULL | NULL    | NULL | 45   | NULL  |
    +----+-------------+-----------+------+---------------+------+---------+------+------+-------+
```

不指定 `explain_type` 或者指定为 `FORMAT = TRADITIONAL` 效果一样。

### 指定 `explain_type`

- `EXTENDED`

```
    DESC extended SELECT * FROM $tb_name\G
    
    --> 输出示例：
    
    id           : 1
    select_type  : SIMPLE
    table        : $tb_name
    type         : ALL
    possible_keys: NULL
    key          : NULL
    key_len      : NULL
    ref          : NULL
    rows         : 45
    filtered     : 100.00
    Extra        : NULL
```

可见，`explain_type` 为 `extended` 时，会多返回一个 `filtered` 属性值。

此外，当指定了 `extended` 参数时，`DESC` 会生成额外的提示信息，可以紧接着通过 `SHOW WARNINGS` 查看。

- `FORMAT = json`

```
    DESC format = json SELECT * FROM $tb_name\G
    
    --> 输出示例：
    
    EXPLAIN: {
      "query_block": {
        "select_id": 1,
        "table": {
          "table_name": "$tb_name",
          "access_type": "ALL",
          "rows": 45,
          "filtered": 100
        }
      }
    }
```

注意，`explain_type` 为 `format = json` 时，`DESC` 的输出已经自动地输出了 `expain_type` 既为 `extened`，又为 `partition` 的信息，可以看到，这里也返回了一个 `filtered` 属性值。（`expain_type` 为 `extened` 的输出）

## explain 输出解释

下面以 **列名/JSON NAME：** 对应含义 的格式进行说明。`JSON_NAME` 指 `format = json` 时的输出字段名。

- `id`/`select_id`

本次 SELECT 查询的序列号。

当本次查询为 `UNION` 类型时，可以为 `NULL` 。此时的 `table` 将显示 `<unionM,N,Q>`，`select_type` 将显示 `UNION RESULT，M、N、Q` 分别代表 `UNION` 中所有查询的序列号。

- `select_type/none`

SELECT 的类型

- `table`/`table_name`：SELECT 的表名。​

# 参考

- [MySQL :: MySQL 5.6 Reference Manual :: 13.8 Utility Statements](https://dev.mysql.com/doc/refman/5.6/en/sql-syntax-utility.html)