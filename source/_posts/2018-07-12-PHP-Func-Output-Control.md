---
title: PHP 函数「输出控制」
date: 2018-07-12 10:15:11
categories: PHP Func
tags: [PHP, Function]
---

> PHP 输出控制函数

<!-- more -->

# `php.net`

- `flush` - 刷新输出缓冲
- `ob_clean` - 清空输出缓冲区
- `ob_end_clean` - 清空缓冲区并关闭输出缓冲
- `ob_end_flush` - 冲刷出输出缓冲区内容并关闭缓冲
- `ob_flush` - 冲刷出输出缓冲区中的内容
- `ob_get_clean` - 得到当前缓冲区的内容并删除当前输出缓存
- `ob_get_content` - 返回输出缓冲区的内容
- `ob_get_flush` - 刷出缓冲区内容, 以字符串形式返回内容, 并关闭输出缓冲区
- `ob_get_length` - 返回输出缓冲区内容的长度
- `ob_get_level` - 返回输出缓冲机制的嵌套级别
- `ob_get_status` - 得到所有输出缓冲区的状态
- `ob_gzhangdler` - 在`ob_start中`使用的用来压缩输出缓冲区中内容的回调函数
- `ob_implicit_flush` - 打开/关闭绝对刷送
- `ob_list_handlers` - 列出所有使用中的输出处理程序
- `ob_start` - 打开输出控制缓冲
- `output_add_rewrite_var` - 添加URL重写器的值
- `output_reset_rewrite_vars` - 重设URL重写器的值

# 常用的输出控制函数说明

## `ob_start`

> 此函数将打开输出缓冲。当输出缓冲激活后，脚本将不会输出内容（除http标头外），相反需要输出的内容被存储在内部缓冲区中。

内部缓冲区的内容可以用 `ob_get_contents()` 函数复制到一个字符串变量中. 
想要输出存储在内部缓冲区中的内容, 可以使用 `ob_end_flush()` 函数。另外， 使用 `ob_end_clean()` 函数会静默丢弃掉缓冲区的内容。

**输出缓冲区是可堆叠的**，这即意谓着，当有一个 `ob_start()` 是活跃的时， 你可以调用另一个 `ob_start()` 。 
只要确保正确调用了 `ob_end_flush()` 恰当的次数即可。 如果有多重输出回调函数是活跃的，输出内容会一直按嵌套的顺序依次通过它们而被过滤。

## `flush`

> 刷新PHP程序的缓冲，该函数将当前为止程序的所有输出发送到用户的浏览器。

`flush()` 函数不会对服务器或客户端浏览器的缓存模式产生影响。
因此，必须同时使用 `ob_flush()` 和 `flush()` 函数来刷新输出缓冲。

## `ob_flush`

> 冲刷出输出缓冲区中的内容

输出缓冲区中的内容，如果想进一步处理缓冲区中的内容，**必须在`ob_flush()`之前调用`ob_get_contents()`** ，
因为在调用`ob_flush()`之后缓冲区内容将被丢弃，而缓冲区不会被销毁。

## `ob_end_flush`

> 输出缓冲区内容，并关闭输出缓冲区。

## `ob_get_flush`

> 输出缓冲区内容(以字符串形式返回)，并关闭输出缓冲区，与`ob_end_flush()`不同的是**本函数还会以字符串形式返回缓冲区内容**。

## `ob_clean`

> 清空输出缓冲区, 此函数用来丢弃输出缓冲区的内容
>
> 此函数不会像 `ob_end_clean()` 函数那样销毁输出缓冲区。
> 
> 输出缓冲必须已被 `ob_start()` 以 `PHP_OUTPUT_HANDLER_CLEANABLE` 标记启动。否则 `ob_clean()` 不会有效果。

## `ob_end_clean`

> 清空输出缓冲区并关闭输出缓冲区; 此函数丢弃最顶层输出缓冲区的内容并关闭这个缓冲区

## `ob_get_clean`

> 得到当前缓冲区的内容并删除当前输出缓存
>
> 返回输出缓冲区的内容，并结束输出缓冲区。如果输出缓冲区不是活跃的，即返回 `FALSE` 。

## `ob_get_conent`

> 获取缓冲区的内容

## `ob_get_length`

> 获取缓冲区内容的长度

## `ob_get_level`

> 获取缓冲机制的嵌套级别

## `ob_get_status`

> 得到所有输出缓冲区的状态

# 实例

## 使用控制输出函数生成静态页面

```php
     <?php
        
        ob_start();
        
        echo "<html><head><title>PHP静态化设计</title></head><body>Hello OB</body></html>";
        
        $out = ob_get_contents();
        
        ob_end_clean();
        
        $fp = fopen("static.html", "w");
        
        if (!$fp) {
            echo "System Error.";exit();
        } else {
            fwrite($fp, $out);
            fclose($fp);
        
            echo "Success";
        }
```
