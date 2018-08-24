---
title: 关于 PJAX
date: 2018-08-23 09:53:42
categories: 前端
tags: [前端, PJAX]
---

> 使用 `AJAX` 和 `window.history.pushState` 无刷新改变页面内容和地址栏 URL。

<!-- more -->

 
 在访问现在很火的 Google plus 时，细心的用户也许会发现页面之间的点击是通过 AJAX 异步请求的，同时页面的URL发生了了改变。并且能够很好的支持浏览器的前进和后退。不禁让人想问，是什么有这么强大的功能呢？
 
 HTML5 里引用了新的API，就是 `history.pushState` 和 `history.replaceState`，就是通过这个接口做到无刷新改变页面 URL 的。

# 与传统的AJAX的区别

传统的 AJAX 有如下的问题：

- 1. 虽然 AJAX 可以无刷新改变页面内容，但无法改变页面URL。
- 2. 其次为了更好的可访问性，内容发生改变后，改变 URL 的 `hash`。但是 `hash` 的方式不能很好的处理浏览器的前进、后退等问题 有的浏览器引入了 `onhashchange` 的接口，不支持的浏览器只能定时去判断 hash 是否改变。
- 3. 再有， AJAX 的使用对搜索引擎很不友好，往往蜘蛛爬到的区域是空的。

为了解决传统 AJAX 带来的问题，HTML5里引入了新的API，即：`history.pushState`, `history.replaceState`。

可以通过 `pushState` 和 `replaceState` 接口操作浏览器历史，并且改变当前页面的URL。

- `pushState` 是将指定的 URL 添加到浏览器历史里;
- `replaceState` 是将指定的 URL 替换当前的 URL。

# 如何调用

```
    var state = {
    	title: title,
    	url: options.url,
    	otherkey: othervalue
    } ;
    window.history.pushState(state, document.title, url);
```

state 对象除了要 title 和 url 之外，也可以添加其他的数据，比如：还想将一些发送 AJAX 的配置给保存起来。

`replaceState` 和 `pushState` 是相似的，不需要多做解释。

# 如何响应浏览器的前进、后退操作?

window 对象上提供了 `onpopstate` 事件，上面传递的 state 对象会成为 event 的子对象，这样就可以拿到存储的 title 和 URL 了。

```
    window.addEventListener('popstate', function(e){
    if (history.state){
    	var state = e.state;
    	// do something(state.url, state.title);
    }
    }, false) ;
```

这样就可以结合 `AJAX` 和 `pushState` 完美的进行无刷新浏览了。

# 一些限制

- 1. 无法跨域，这个是必然的。引用曾经在网上看到的一句经典的话：
> 如果javascript可以跨域的话，那他就可以逆天了…

- 2. state 对象虽然可以存储很多自定义的属性，但值不能是个对象

# 对应后端的一些处理

这种模式下除了当前使用 AJAX 可以无刷新浏览外，还要保证直接请求改变的URL后也可以正常浏览，所以后端要对这些处理下。

- 1. 对结合 `pushState` 的 AJAX 可以发送一个特殊的头，如：`setRequestHeader(‘PJAX’, ‘true’)`。

- 2. 后端获取到有 `PJAX=true` 的 header 时，将页面中通用的部分都不输出。

比如：PHP 可以通过下面的判断:

```
    function is_pjax(){
    	return array_key_exists('HTTP_X_PJAX', $_SERVER) && $_SERVER['HTTP_X_PJAX'] === 'true';
    }
```

虽然接口上只有 `pushState`、`replaceState`、`onpopstate`，但在使用的时候还是要做很多处理的。

# 快捷使用PJAX

- 引入PJAX

```
    <script src='https://cdnjs.cloudflare.com/ajax/libs/jquery.pjax/2.0.1/jquery.pjax.min.js'></script>
```

- 调用PJAX

```
     $(document).pjax('a[href^="http://blog.caoxl.com"]:not(a[target="_blank"], a[no-pjax])', {
            container: '#page', // 需要刷新的标签
            fragment: '#page',  // 容器 ???
            timeout: 8000
     });
```