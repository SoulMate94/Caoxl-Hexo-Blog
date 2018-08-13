---
title: XML与Array互相转换
date: 2018-06-27 09:52:03
categories: PHP
tags: PHP
---

> 封装XML和Array互相转换的方法

<!-- more -->

# XML2Array

```
    public static function xml2Array(string $xml)
    {
        return json_decode(json_encode(simplexml_load_string(
            $xml,
            'SimpleXMLElement',
            LIBXML_NOCDATA
        )), true);
    }
```

## 说明

- `json_encode` — 对变量进行 `JSON` 编码
- `json_decode` — 对 `JSON` 格式的字符串进行解码,当该`assoc`为 `TRUE` 时，将**返回 `array` 而非 `object`**
- `simplexml_load_string` - 将格式良好的XML字符串将其作为对象返回

# Array2XML

```
    public static function array2XML(array $array, string &$xml): string
    {
        foreach ($array as $key => &$val) {
            if (is_array($val)) {
                $_xml = '';
                $val  = self::array2XML($val, $_xml);
            }
            $xml .= "<$key>$val</$key>";            
        }
        
        unset($val);
        
        return $xml;
    }
```

## 说明

- `: string` 写法是`PHP7`的新特性


# 参考

- [LIBXML预定义常量](https://secure.php.net/manual/zh/libxml.constants.php)
