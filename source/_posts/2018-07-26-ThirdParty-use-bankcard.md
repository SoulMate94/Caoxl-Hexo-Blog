---
title: 根据银行卡号获取银行信息 「阿里API」
date: 2018-07-26 11:17:02
categories: ThirdParty
tags: [PHP, 第三方, ThirdParty, 银行卡]
---

> 根据银行卡号获取银行信息（银行名称, 信用卡/借记卡, 银行LOGO 等）, 供任何 PHP 框架或者原生代码使用.

<!-- more -->

还记得在支付宝上给同学同志同事女朋友打钱的时候，当你输入完银行卡号的时候自动帮你选择好银行卡的小细节吗？当你给信用卡还款的时候，能自动判断出是信用卡还是储蓄卡。如此贴心的功能，你值得拥有！

```
    // 只输入前五位即可查询
    BankCard::info('6225700000000000');
    
    // 将得到
    array (size=6)
      'validated'    => true            // 是否验证成功
      'bank'         => 'CEB',          // 银行标识
      'bankName'     => '中国光大银行' ,  // 银行名称
      'bankImg'      => 'https://apimg.alipay.com/combo.png?d=cashier&t=CEB',  // 银行LOGO
      'cardType'     => 'CC',           // 卡类型
      'cardTypeName' => '信用卡',        // 卡类型名称
```

# 引入

## `Laravel`/`Lumen`

```
    composer require zhuzhichao/bank-card-info
```

## 其他框架

> 点击下载: [zhuzhichao/bank-card-info](https://packagist.org/packages/zhuzhichao/bank-card-info)

# 使用

- 超级简单

```
    use Zhuzhichao\BankCardInfo\BankCard;
    
    $bankInfo = BankCard::info($bank_card_number);
    
    var_dump($bankInfo);
```

## 单独获取银行logo

- `BankCard::getBankImg('ABC')`

```
    https://apimg.alipay.com/combo.png?d=cashier&t=ABC
```

## 获取银行列表信息

- `BankCard::getBankList()`

```
    array (size=165)
      'SRCB'   =>  '深圳农村商业银行',
      'BGB'    =>  '广西北部湾银行',
      'SHRCB'  =>  '上海农村商业银行',
      'BJBANK' =>  '北京银行',
      'WHCCB'  =>  '威海市商业银行',
      'BOZK'   =>  '周口银行',
      ...
      'LYBANK' =>  '洛阳银行',
      'GDB'    =>  '广东发展银行',
      'ZBCB'   =>  '齐商银行',
      'CBKF'   =>  '开封市商业银行',
      ...
```

# 特点

- 1. 不配置和使用数据库，妈妈再也不用担心配置问题了
- 2. 使用简单，功能专(dān)注(yī)
- 3. 使用`composer`进行安装管理，国际标准，方便快捷，即安即用，随时更新数据库

> 该接口由支付宝提供~~~

# 参考

- [Bank card info](https://github.com/zhuzhichao/bank-card-info)
