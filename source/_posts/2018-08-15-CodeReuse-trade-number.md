---
title: 「代码复用」 生成订单编号
date: 2018-08-15 15:34:37
categories: 代码复用
tags: [CodeReuse, 订单编号, tradeNo]
---

> 这里生成的是仿造支付宝订单号

<!-- more -->

```
    <?php
    
    namespace App\Traits;
    
    class Tool
    {
        // Generate inner system trade number
        // $uid: user id
        // $from: 01 => user; 02 => shop; 03 => staff; 04 => refund; ...
        // $domain: 00 => master
        // $suffix: 后缀
        // @caoxl
        public static function tradeNo(
            $uid = 0,
            $from = '01',
            $domian = '00'
        ): string
        {
            $domian  = str_pad(($domian%42), 2, '0', STR_PAD_LEFT);
            $uid     = str_pad(($uid%1024), 4, '0', STR_PAD_LEFT);
            $from    = in_array($from, ['01', '02', '03']) ? $from : '00';
            $suffix  = mb_substr(microtime(), 2, 6);
        
            return date('YmdHis').$domian.$from.$uid.mt_rand(1000, 9999).$suffix;
        }
    }
```

> 例如: 20180815074921000101004317335415

- `20180815074608` - `date('YmdHis)`
- `00` - 处理后的`master`
- `01` - `from user`
- `0001` - 处理后的`uid`
- `4317` - `mt_rand(1000, 9999)`
- `335415` - `mb_substr(microtime(), 2, 6)`


**说明:**

- `str_pad` — 使用另一个字符串填充字符串为指定长度

```
    str_pad($input, $pad_length, pad_string, [pad_type])
    
    // 可选的 pad_type 参数的可能值为 STR_PAD_RIGHT，STR_PAD_LEFT 或 STR_PAD_BOTH。如果没有指定 pad_type，则假定它是 STR_PAD_RIGHT。
```

