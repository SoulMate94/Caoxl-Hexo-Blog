---
title: PHP 中的空元素和 NULL
date: 2018-02-01 16:56:52
categories: PHP
tags: [PHP, NULL]
---

变量为空和值为 NULL 的变量是各大编程语言中都容易混淆的概念，今天就总结一下 PHP 中的空元素和 NULL 常量。

<!--more-->

# 运行


```
    <?php
    
    /**
     * 测试 PHP 中的 null, false, '0', '', 0, array(), var $var 是否：相等、全等、为空、为 NULL
     * 没有任何属性的元素都为空
     * NULL 类型只有一个值，就是不区分大小写的特殊常量 NULL，表示变量没有值
     * 在下列情况下一个变量被认为是 NULL：被赋值为 NULL；尚未被赋值；被 unset()
     * 使用 `(unset) $var` 将一个变量转换为 null 不会删除该变量或 unset 其值，仅是返回 NULL 值而已
     */
    $cpr = array(
        'null'    => null,
        'false'   => false,
        '0'       => 0,
        "''"      => '',
        "'0'"     => '0',
        'array()' => array(),
        '$var'    => @$var
    );
    
    $len = count( $cpr ) ;
    
    foreach ( $cpr as $k=>$v ) {
        $keys[] = $k ;
        $vals[] = $v ;
    }
    echo '在 PHP 中:'."\n\n" ;
    
    for ( $i=0; $i<$len; ++$i ) {
        echo "\n", $keys[ $i ] ;
        echo ( empty( $vals[ $i ] ) ) ? ' 为空' : ' 不为空' ;
        echo ( is_null( $vals[ $i ] ) ) ? ' 且为 NULL' : ' 但不为 NULL' ;
        echo "\n" ;
    
        for ( $j=$i+1; $j<$len; ++$j ) {
            if ( $vals[ $i ] == $vals[ $j ] ) {
                if ( !($vals[ $i ] === $vals[ $j ]) ) {
                    echo "\n\t", $keys[ $i ], ' 和 ', $keys[ $j ], ' 的值相等, 但是类型不相等', "\n" ;
                }
            } else {
                echo "\n\t", $keys[ $i ], ' 和 ', $keys[ $j ], ' 不相等', "\n" ;
            }
            echo "\n" ;
        }
    }
```

# 总结

```
    在 PHP 中:
    
    
    null 为空 且为 NULL
    
    	null 和 false 的值相等, 但是类型不相等
    
    
    	null 和 0 的值相等, 但是类型不相等
    
    
    	null 和 '' 的值相等, 但是类型不相等
    
    
    	null 和 '0' 不相等
    
    
    	null 和 array() 的值相等, 但是类型不相等
    
    
    
    false 为空 但不为 NULL
    
    	false 和 0 的值相等, 但是类型不相等
    
    
    	false 和 '' 的值相等, 但是类型不相等
    
    
    	false 和 '0' 的值相等, 但是类型不相等
    
    
    	false 和 array() 的值相等, 但是类型不相等
    
    
    	false 和 $var 的值相等, 但是类型不相等
    
    
    0 为空 但不为 NULL
    
    	0 和 '' 的值相等, 但是类型不相等
    
    
    	0 和 '0' 的值相等, 但是类型不相等
    
    
    	0 和 array() 不相等
    
    
    	0 和 $var 的值相等, 但是类型不相等
    
    
    '' 为空 但不为 NULL
    
    	'' 和 '0' 不相等
    
    
    	'' 和 array() 不相等
    
    
    	'' 和 $var 的值相等, 但是类型不相等
    
    
    '0' 为空 但不为 NULL
    
    	'0' 和 array() 不相等
    
    
    	'0' 和 $var 不相等
    
    
    array() 为空 但不为 NULL
    
    	array() 和 $var 的值相等, 但是类型不相等
    
    
    $var 为空 且为 NULL
```
