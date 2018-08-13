---
title: PHP 加密解密
date: 2018-07-13 09:30:11
categories: PHP
tags: [PHP]
---

> 加密不同于密码，加密是一个动作或者过程，其目的就是将一段明文信息（人类或机器可以直接读懂的信息）变为一段看上去没有任何意义的字符，必须通过事先约定的解密规则才能将信息转换回有意义的可读信息，通过加密可以防止非授权的信息窃取。

<!-- more -->

# 存储加密和传输加密

- 存储加密

> 存储加密是指对存储在纸质、磁盘、数据库等介质上的数据进行的加密

- 传输加密

> 传输加密则是指对数据在计算机网络、电话、电报等通信信道上进行的加密

**不管是上述哪种加密，本质上仍是对信息进行加密。**

# 加密算法

## 对称加密

> 所谓对称加密，就是加密和解密使用同一秘钥，这也是这种加密算法最显著的缺点之一。
>
> 现代的加密算法中，DES、3DES、AES等算法都属于对称加密算法
>
> 对称加密有一个明显的缺点，就是即秘钥。特别是在传输加密时，信息的发送方和接收方需要使用相同的秘钥来对信息进行加解密，接收方如何安全的获取秘钥成为这类加密的焦点，因为一旦秘钥被截获，整个加密通信就形同明文传输。
>

**对称加密比较适合存储加密**

## 非对称加密

> 这种加密算法的秘钥分为“公开秘钥”和“私有秘钥”，公开秘钥用于对信息进行加密，而解密时使用私有秘钥进行解密，这样，信息的接收方可以事先生成好一份公钥和私钥，然后将公钥发给所有的信息发送方，信息发送方使用公钥对信息进行加密，然后将信息发送给接收方，接收方使用私钥进行解密即可
> 
> 这种算法的优势在于，解密的私钥不需要传递，降低（只能降低，无法避免，要考虑认为因素）了私钥泄密的可能性
>
> 常见的非对称加密算法有：RSA、EIGamal、背包算法、Rebin（RSA的特例）、迪菲－赫尔曼密钥交换协议中的公钥加密算法和椭圆曲线加密算法等
>
> 而最为大家熟知的就是RSA算法

**其主要缺点之一就是慢，适合加密少量数据。**

## 误区

看到这里，有些人可能会问，我常用的`md5`、`hash算法`(`sha1`、`sha256`、`sha512`、`sha1024`等）怎么没见你提及

> 其实准确的说，md5和hash算法不能算是加密算法，它们都属于信息摘要算法，可以为不同的信息生成独一无二的信息摘要，而它们都属于不可逆算法，即无法通过生成的摘要信息还原出原始信息

# PHP加密最佳实践

加密总是与安全密不可分，而每个`PHPer`都必须将应用安全作为必要的设计思路融入代码中，以下是一些最佳实践的建议。

- 1. 不要再使用`MD5`，不要使用`sha1`，基本上已经没有破解难度了
- 2. 请使用`password_hash`来哈希密码(php版本大于等于5.5，小于5.5请使用`password_compat`库)，由于`password_hash`函数已帮你处理好了加盐，而且作为盐的随机字串已通过加密算法成为了哈希的一部分，`password_verify()`函数会自动将盐从哈希中提取出来，所以你无需考虑盐的存储问题。
- 3. 通信接口的签名，请使用非对称算法对签名秘钥进行加密，并对秘钥设置有效期，定期更换。

> 不过还是需要了解加密与解密函数的~

# 加密函数

## `MD5`

> `md5()`默认情况下以 `32` 字符十六进制数字形式返回散列值，它接受两个参数，第一个为**要加密的字符串**，第二个为**`raw_output`的布尔值**，默认为`false`，如果设置为`true`，`md5()`则会返回原始的 `16` 位二进制格式报文摘要
>
> `md5()`为单向加密，没有逆向解密算法，但是还是可以对一些常见的字符串通过收集，枚举，碰撞等方法破解

```
    $username = 'caoxl';
    $password = 'caoxl.com';
    
    echo md5($username);
    echo '<hr>';
    echo md5($password);
    echo '<hr>';
    echo md5(md5($password));
    
    # 得到
    // 39e1e7e20baad16de47d95d155dc84d8
    // 7badf979f092ff575c62bd85f2fbb06d
    // b380ba1215610227ad957856d5a8574c
```

## `Crypt`

> `crypt()`接受两个参数，第一个为**需要加密的字符串**，第二个为**盐值**（就是加密干扰值，如果没有提供，则默认由PHP自动生成); 返回散列后的字符串或一个少于 `13` 字符的字符串，后者为了区别盐值。
>
> `crypt()`为单向加密，跟`md5`一样。

```
    $password = 'caoxl.com';
    echo crypt($password);
    
    echo '<hr>';
    echo crypt($password, 'caoxl');
    
    echo '<hr>';
    echo crypt($password, '$1$caoxl$');
```

## `Sha1`

> 跟`md5`很像，不同的是`sha1()`默认情况下返回`40`个字符的散列值，传入参数性质一样，第一个为加密的字符串，第二个为`raw_output`的布尔值，默认为`false`，如果设置为`true`，`sha1()`则会返回原始的`20` 位原始格式报文摘要
>
> `sha1()`也是单行加密，没有逆向解密算法

```
    $username = 'caoxl';
    echo sha1($username);
    
    echo '<hr>';
    echo md5(sha1($username));
```

## `Sha2` - `SHA256`/`SHA512`

> 这两个加密方式分别生成`256`和`512`比特长度的`hash`字串。

> `PHP`内置了`hash()`函数，你只需要将加密方式传给`hash()`函数就好了。你可以直接指明`sha256`, `sha512`, `md5`, `sha1`等加密方式。

```
    $string    = 'qadbk+365t';
    $password1 = hash("sha256", $string);
    echo $password1;
    // f07e1599e5cd19c27fc6968997c09b95a4fc8178581ebab6397aa2341b233f38
    echo '<hr>';
    
    
    $password2 = hash("sha512", $string);
    echo $password2;
    // ddf4d3860b5b0cc7adcfc61bed0e9cee2e92a849ce6a48d7e638fcc465f6e74940867e1e161ba3acaf5698c0bdad3ea0bad6a989d39bedd9f70f26d5f9f507fc
    echo '<hr>';
    
    $password3 = hash("md5", $string);
    echo $password3;
    // 6a175a68affe2e511b4a2f35bdf52b63
    echo '<hr>';
    
    $password4 = hash("sha1", $string);
    echo $password4;
    // 230b01dbc74306bdd0e7826ae5b3461a3ed6374e
```

## `Urlencode`

> 一个参数，传入要加密的字符串 (通常应用于对URL的加密)
> 
> `urlencode`为双向加密，可以用`urldecode`来加密(严格意义上来说，不算真正的加密)
>
> 返回字符串，此字符串中除了 `-_.` 之外的所有非字母数字字符都将被替换成百分号(`%`) 后跟两位十六进制数，空格则编码为加号(`+`)。

```
    $my_urlencode = "caoxl.com?caoxl=true + 4-3%5= \& @!";
    echo urlencode($my_urlencode);
    // caoxl.com%3Fcaoxl%3Dtrue+%2B+4-3%255%3D+%5C%26+%40%21
    
    echo '<hr>';
    $my_urldecode="caoxl.com%3Fcaoxl%3Dtrue+%2B+4-3%255%3D+%5C%26+%40%21";
    echo urldecode($my_urldecode);
    // caoxl.com?caoxl=true + 4-3%5= \& @!
```

### 常见的`urlencode`转换字符

```
    ? => %3F
    = => %3D
    % => %25
    & => %26
    \ => %5C
```

## `base64`

> `base64_encode()`接受一个参数，也就是要编码的数据(这里不说字符串，是因为很多时候`base64`用来编码图片)
>
> `base64_encode()`为双向加密，可用`base64_decode()`来解密

```
    $imgUrl = 'https://caoxl.com/images/php7.jpg';
    echo base64_encode($imgUrl);
    // aHR0cHM6Ly9jYW94bC5jb20vaW1hZ2VzL3BocDcuanBn
    
    echo '<hr>';
    echo base64_decode('aHR0cHM6Ly9jYW94bC5jb20vaW1hZ2VzL3BocDcuanBn');
    // https://caoxl.com/images/php7.jpg
```

## `Password Hashing API`

`Password Hashing API`是`PHP 5.5`之后才有的新特性，它主要是提供下面几个函数供我们使用

- `password_hash()` - 对密码加密
- `password-verify()` - 验证已经加密的密码，检验其hash字串是否一致
- `password_needs_rehash()` - 给密码重新加密
- `password_get_info()` - 返回加密算法的名称和一些相关信息

```php
    <?php
    
    # password_hash
    $password = 'qadbk+365t';
    $hash1    = password_hash($password, PASSWORD_DEFAULT);
    echo $hash1;
    echo '<hr>';
    
    $options = [
        'salt' => md5('caoxl'),
        'cost' => 12 // 默认10
    ];
    $hash2 = password_hash($password, PASSWORD_DEFAULT, $options);
    echo $hash2;
    echo '<hr>';
    
    # password_verify
    if (password_verify($password, $hash2)) {
        echo "验证成功";
    } else {
        echo "验证失败";
    }
    echo '<hr>';
    
    # password_needs_rehash
    if (password_needs_rehash($hash2, PASSWORD_DEFAULT, ['cost' => 14])) {
        // cost change to 12
        $hash = password_hash($password, PASSWORD_DEFAULT, ['cost' => 14]);
    
        echo $hash;
        // don't forget to store the new hash!
    }
    
    # password_get_info
    print_r( password_get_info( $hash2 ) );
    // Array ( [algo] => 1 [algoName] => bcrypt [options] => Array ( [cost] => 12 ) )
```

- `password_get_info()` 返回信息
  - `algo` - 匹配[密码算法的常量](https://secure.php.net/manual/zh/password.constants.php)
  - `algoName` - 算法名字
  - `options` - 加密时候的可选参数


# 参考

- [PHP的几个常用加密函数](https://jellybool.com/post/php-encrypt-functions)