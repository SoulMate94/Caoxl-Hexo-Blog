---
title: 「代码复用」数组与XML互转
date: 2018-08-08 09:59:58
categories: CodeReuse
tags: [代码复用, XML, 数组]
---

> Don't repeat yourself

<!-- more -->

# `xmlToArray`

```
    public static function xmlToArray(string $xml)
    {
        return json_decode(json_encode(simplexml_load_string(
            $xml,
            'SimpleXMLElement',
            LIBXML_NOCDATA
        )), true);
    }
```

- `json_decode()`

> 对 JSON 格式的字符串进行解码,当该assoc为 TRUE 时，将返回 array 而非 object

- `json_encode()`

> json_encode — 对变量进行 JSON 编码

- `simplexml_load_string()`

> simplexml_load_string - 将格式良好的XML字符串将其作为对象返回

# `arrayToXML`

```
    public static function arrayToXML(array $array, string &$xml): string
    {
        foreach ($array as $key => &$val) {
            if (is_array($val)) {
                $_xml = '';
                $val = self::arrayToXML($val, $_xml);
            }
            $xml .= "<$key>$val</$key>";
        }

        unset($val);

        return $xml;
    }
```

> : string 写法是PHP7的新特性
