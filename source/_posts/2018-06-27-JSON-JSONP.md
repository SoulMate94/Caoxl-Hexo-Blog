---
title: 说说JSON和JSONP
date: 2018-06-27 14:49:12
categories: www
tags: [www,JSON,JSONP]
---

> `JSON`和`JSONP`虽然只有一个字母的差别，但其实他们根本不是一回事儿;
就像Java和JavaScript没有关系,老婆和老婆饼没有关系一个道理

<!-- more -->

# 什么是`JSON`?

> `JSON`是一种基于文本的数据交换方式，或者叫做数据描述格式

## `JSON`的优点

1. 基于纯文本，跨平台传递极其简单
2. `Javascript`原生支持，后台语言几乎全部支持
3. 轻量级数据格式，占用字符数量极少，特别适合互联网传递
4. 可读性较强，虽然比不上`XML`那么一目了然，但在合理的依次缩进之后还是很容易识别的
5. 容易编写和解析，当然前提是你要知道数据结构

## `JSON`的格式

`JSON`能够以非常简单的方式来描述数据结构，XML能做的它都能做，因此在跨平台方面两者完全不分伯仲。

1. `JSON`只有两种数据类型描述符，**大括号{}**和**方括号[]**，其余英文冒号`:`是映射符，英文逗号`,`是分隔符，英文双引号`""`是定义符。
2. 大括号`{}`用来描述一组“不同类型的无序键值对集合”（每个键值对可以理解为`OOP`的属性描述），方括号[]用来描述一组“相同类型的有序数据集合”（可对应`OOP`的数组）
3. 上述两种集合中若有多个子项，则通过英文逗号`,`进行分隔。
4. 键值对以英文冒号`:`进行分隔，并且建议键名都加上英文双引号`""`，以便于不同语言的解析。
5. `JSON`内部常用数据类型无非就是`字符串`、`数字`、`布尔`、`日期`、`null` 这么几个，字符串必须用双引号引起来，其余的都不用，日期类型比较特殊，这里就不展开讲述了

## 实例

```
    {
        "Name": "caoxl",
        "Age": 23,
        "Com": "IBM"
    },
    {
        "Name": "caolx",
        "Age": 24,
        "Com": "IBM"
    }
```

## `JSON`

```
    $.ajax({
        type: "post",
        url: "danmu.php",
        data: {word:"adc",username:"cxl"},
        datatype: "json",
        async: true, //是否异步,true为异步
        
        // success为数据加载完成后的回调函数
        success: function (data) {
            var show = document.getElementById('show');
            for (i in data) {
                show.innerHTML += data[i]+"<br>;
            }
            console.log(data);
        }
    });
```

# 什么是`JSONP`?

> `JSONP（JSON with Padding）`是数据格式JSON的一种“使用模式”，可以让网页从别的网域要数据。

## 先说说`JSONP`是怎么产生的：

1. `Ajax`直接请求普通文件存在跨域无权限访问的问题
2. `Web`页面上调用`js`文件时则不受是否跨域的影响（不仅如此，我们还发现凡是拥有`"src"`这个属性的标签都拥有跨域的能力，比如`<script>`、`<img>`、`<iframe>`）
3. 跨域访问数据就只有一种可能，那就是在远程服务器上设法把数据装进`js`格式的文件里，供客户端调用和进一步处理
4. 恰巧我们已经知道有一种叫做`JSON`的纯字符数据格式可以简洁的描述复杂数据，更妙的是`JSON`还被`js`原生支持
5. `Web`客户端通过与调用脚本一模一样的方式，来调用跨域服务器上动态生成的`js`格式文件（一般以`JSON`为后缀），
显而易见，服务器之所以要动态生成`JSON`文件，目的就在于把客户端需要的数据装入进去
6. 客户端在对`JSON`文件调用成功之后，也就获得了自己所需的数据，剩下的就是按照自己需求进行处理和展现了，这种获取远程数据的方式看起来非常像`AJAX`，但其实并不一样
7. 为了便于客户端使用数据，逐渐形成了一种**非正式传输协议**，人们把它称作`JSONP`，该协议的一个要点就是允许用户传递一个`callback`参数给服务端，
然后服务端返回数据时会将这个`callback`参数作为函数名来包裹住`JSON`数据，这样客户端就可以随意定制自己的函数来自动处理返回数据了。

## `JSONP`的客户端具体实现

1. 我们知道，哪怕跨域js文件中的代码（当然指符合web脚本安全策略的），web页面也是可以无条件执行的。

- 远程服务器`remoteserver.com`根目录下有个`remote.js`文件代码如下：

```
    alert('我是远程文件,跨域调用成功');
```

- 本地服务器`localserver.com`下有个`jsonp.html`页面代码如下：

```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title></title>
        <script type="text/javascript" src="http://remoteserver.com/remote.js"></script>
    </head>
    <body>
    </body>
    </html>
```

毫无疑问，页面将会弹出一个提示窗体，显示跨域调用成功。

2. 现在我们在`jsonp.html`页面定义一个函数，然后在远程`remote.js`中传入数据进行调用。

```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title></title>
        <script type="text/javascript">
        var localHandler = function(data){
            alert('我是本地函数，可以被跨域的remote.js文件调用，远程js带来的数据是：' + data.result);
        };
        </script>
        <script type="text/javascript" src="http://remoteserver.com/remote.js"></script>
    </head>
    <body>
    
    </body>
    </html>
```

3. 聪明的开发者很容易想到，只要服务端提供的js脚本是动态生成的就行了呗，这样调用者可以传一个参数过去告诉服务端“我想要一段调用XXX函数的js代码，请你返回给我”，于是服务器就可以按照客户端的需求来生成js脚本并响应了。

```html
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title></title>
        <script type="text/javascript">
        // 得到航班信息查询结果后的回调函数
        var flightHandler = function(data){
            alert('你查询的航班结果是：票价 ' + data.price + ' 元，' + '余票 ' + data.tickets + ' 张。');
        };
        // 提供jsonp服务的url地址（不管是什么类型的地址，最终生成的返回值都是一段javascript代码）
        var url = "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998&callback=flightHandler";
        // 创建script标签，设置其属性
        var script = document.createElement('script');
        script.setAttribute('src', url);
        // 把script标签加入head，此时调用开始
        document.getElementsByTagName('head')[0].appendChild(script); 
        </script>
    </head>
    <body>
    
    </body>
    </html
```


4. 到这里为止的话，相信你已经能够理解`jsonp`的客户端实现原理了吧？剩下的就是如何把代码封装一下，以便于与用户界面交互，从而实现多次和重复调用。

### 想知道`jQuery`如何实现`jsonp`调用？

```
     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
     <html xmlns="http://www.w3.org/1999/xhtml" >
     <head>
         <title>Untitled Page</title>
          <script type="text/javascript" src=jquery.min.js"></script>
          <script type="text/javascript">
         jQuery(document).ready(function(){ 
            $.ajax({
                 type: "get",
                 async: false,
                 url: "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998",
                 dataType: "jsonp",
                 jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
                 jsonpCallback:"flightHandler",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
                 success: function(json){
                     alert('您查询到航班信息：票价： ' + json.price + ' 元，余票： ' + json.tickets + ' 张。');
                 },
                 error: function(){
                     alert('fail');
                 }
             });
         });
         </script>
         </head>
      <body>
      </body>
     </html>
```

## `JSONP`

```
    <script type="text/javascript>
        // 自定义的回调函数
        function show(val) {
            document.getElementById("show").src=val[0].src;
        }
        $.ajax({
            type: "get",
            url: "myphp.php",
            datatype: "jsonp",
            jsonp: "callmyapp",
            jsonpCallback: "show",
            async: true,
        });
    </script>
```

# 总结

只有必须要跨域请求时才使用`JSONP`。

# 参考

- [【原创】说说JSON和JSONP，也许你会豁然开朗，含jQuery用例](https://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)
- [json与jsonp的区别](https://www.jianshu.com/p/f46dd756873e)