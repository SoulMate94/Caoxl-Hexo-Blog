---
title: 偶尔看看前端 「CSS3」
date: 2018-07-23 16:00:26
categories: 前端
tags: [前端, CSS3, FrontEnd]
---

> `CSS3` 是最新的 `CSS` 标准。 作为一个后端还是要偶尔看看前端的.
CSS是入门PHP的时候该看的, 为啥现在发这个呢, --因为人都需要不断的学习和复习.

<!-- more -->

# `CSS3`模块

- 选择器
- 框模型
- 背景和边框
- 文本效果
- 2D/3D转换
- 动画
- 多列布局
- 用户界面

# `CSS3`边框 

## CSS3 圆角边框

```
    div {
        border:2px solid;
        border-radius:25px;
        -moz-border-radius:25px; /* Old Firefox */
    }
```

## CSS3 边框阴影

```
    div {
        box-shadow: 10px 10px 5px #888888;
    }
```

## CSS3 边框图片

```
    div {
        border-image:url(border.png) 30 30 round;
        -moz-border-image:url(border.png) 30 30 round; /* 老的 Firefox */
        -webkit-border-image:url(border.png) 30 30 round; /* Safari 和 Chrome */
        -o-border-image:url(border.png) 30 30 round; /* Opera */
    }
```

## 新的边框属性

- `border-image` 设置所有 `border-image-*` 属性的简写属性。
- `border-redius` 设置所有四个 `border-*-radius` 属性的简写属性。
- `box-shadow` 向方框添加一个或多个阴影。


# `CSS3`背景

## `CSS3 background-size` 属性

background-size 属性规定背景图片的尺寸。

- 调整背景图片的大小

```
    div {
        background:url(bg_flower.gif);
        -moz-background-size:63px 100px; /* 老版本的 Firefox */
        background-size:63px 100px;
        background-repeat:no-repeat;
    }
```

- 对背景图片进行拉伸，使其完成填充内容区域

```
    div {
        background:url(bg_flower.gif);
        -moz-background-size:40% 100%; /* 老版本的 Firefox */
        background-size:40% 100%;
        background-repeat:no-repeat;
    }
```

## `CSS3 background-origin` 属性

- 在 `content-box` 中定位背景图片:

```
    div {
        background:url(bg_flower.gif);
        background-repeat:no-repeat;
        background-size:100% 100%;
        -webkit-background-origin:content-box; /* Safari */
        background-origin:content-box;
    }
```

## CSS3 多重背景图片

- 为 body 元素设置两幅背景图片

```
    body { 
        background-image:url(bg_flower.gif),url(bg_flower_2.gif);
    }
```

# CSS3 文本效果

## CSS3 文本阴影

- 向标题添加阴影

```
    h1 {
        text-shadow: 5px 5px 5px #FF0000;
    }
```

## CSS3 自动换行

- 允许对长单词进行拆分，并换行到下一行

```
    p {word-wrap:break-word;}
```

## 新的文本属性

- `hanging-punctuation` - 规定标点字符是否位于线框之外
- `punctuation-trim` - 	规定是否对标点字符进行修剪
- `text-align-last` - 设置如何对齐最后一行或紧挨着强制换行符之前的行
- `text-emphasis` - 向元素的文本应用重点标记以及重点标记的前景色
- `text-justify` - 规定当 text-align 设置为 "justify" 时所使用的对齐方法
- `text-outline` - 	规定文本的轮廓。
- `text-overflow` - 规定当文本溢出包含元素时发生的事情
- `text-shadow` - 向文本添加阴影
- `text-wrap` - 规定文本的换行规则
- `word-break` - 规定非中日韩文本的换行规则
- `word-wrap` - 允许对长的不可分割的单词进行分割并换行到下一行


# CSS3 2D 转换

## `translate()` 方法

> 通过 translate() 方法，元素从其当前位置移动，根据给定的 left（x 坐标） 和 top（y 坐标） 位置参数：

```
    div {
        transform: translate(50px,100px);
        -ms-transform: translate(50px,100px);		/* IE 9 */
        -webkit-transform: translate(50px,100px);	/* Safari and Chrome */
        -o-transform: translate(50px,100px);		/* Opera */
        -moz-transform: translate(50px,100px);		/* Firefox */
    }
```

## `rotate()` 方法

> 通过 rotate() 方法，元素顺时针旋转给定的角度。允许负值，元素将逆时针旋转。

```
    div {
        transform: rotate(30deg);
        -ms-transform: rotate(30deg);		/* IE 9 */
        -webkit-transform: rotate(30deg);	/* Safari and Chrome */
        -o-transform: rotate(30deg);		/* Opera */
        -moz-transform: rotate(30deg);		/* Firefox */
    }
```

## `scale()` 方法

> 通过 scale() 方法，元素的尺寸会增加或减少，根据给定的宽度（X 轴）和高度（Y 轴）参数

```
    div {
        transform: scale(2,4);
        -ms-transform: scale(2,4);	/* IE 9 */
        -webkit-transform: scale(2,4);	/* Safari 和 Chrome */
        -o-transform: scale(2,4);	/* Opera */
        -moz-transform: scale(2,4);	/* Firefox */
    }
```

## `skew()` 方法

> 通过 skew() 方法，元素翻转给定的角度，根据给定的水平线（X 轴）和垂直线（Y 轴）参数

```
    div {
        transform: skew(30deg,20deg);
        -ms-transform: skew(30deg,20deg);	/* IE 9 */
        -webkit-transform: skew(30deg,20deg);	/* Safari and Chrome */
        -o-transform: skew(30deg,20deg);	/* Opera */
        -moz-transform: skew(30deg,20deg);	/* Firefox */
    }
```

## `matrix()` 方法

> 如何使用 matrix 方法将 div 元素旋转 30 度

```
    div {
        transform:matrix(0.866,0.5,-0.5,0.866,0,0);
        -ms-transform:matrix(0.866,0.5,-0.5,0.866,0,0);		/* IE 9 */
        -moz-transform:matrix(0.866,0.5,-0.5,0.866,0,0);	/* Firefox */
        -webkit-transform:matrix(0.866,0.5,-0.5,0.866,0,0);	/* Safari and Chrome */
        -o-transform:matrix(0.866,0.5,-0.5,0.866,0,0);		/* Opera */
    }
```

# CSS3 3D 转换

## `rotateX()` 方法

> 通过 rotateX() 方法，元素围绕其 X 轴以给定的度数进行旋转

```
    div {
        transform: rotateX(120deg);
        -webkit-transform: rotateX(120deg);	/* Safari 和 Chrome */
        -moz-transform: rotateX(120deg);	/* Firefox */
    }
```

## `rotateY()` 旋转

> 通过 rotateY() 方法，元素围绕其 Y 轴以给定的度数进行旋转

```
    div {
        transform: rotateY(130deg);
        -webkit-transform: rotateY(130deg);	/* Safari 和 Chrome */
        -moz-transform: rotateY(130deg);	/* Firefox */
    }
```


# CSS3 过渡

> CSS3 过渡是元素从一种样式逐渐改变为另一种的效果。

要实现这一点, 必须规定两项内容:

- 规定你希望把添加到哪个CSS属性上
- 规定效果的时长

- 应用于宽度属性的过渡效果，时长为 2 秒

```
    div {
        transition: width 2s;
        -moz-transition: width 2s;	/* Firefox 4 */
        -webkit-transition: width 2s;	/* Safari 和 Chrome */
        -o-transition: width 2s;	/* Opera */
    }
```

# CSS3 动画

> 动画是使元素从一种样式逐渐变化为另一种样式的效果

- 当动画为 25% 及 50% 时改变背景色，然后当动画 100% 完成时再次改变

```
    @keyframes myfirst
    {
    0%   {background: red;}
    25%  {background: yellow;}
    50%  {background: blue;}
    100% {background: green;}
    }
    
    @-moz-keyframes myfirst /* Firefox */
    {
    0%   {background: red;}
    25%  {background: yellow;}
    50%  {background: blue;}
    100% {background: green;}
    }
    
    @-webkit-keyframes myfirst /* Safari 和 Chrome */
    {
    0%   {background: red;}
    25%  {background: yellow;}
    50%  {background: blue;}
    100% {background: green;}
    }
    
    @-o-keyframes myfirst /* Opera */
    {
    0%   {background: red;}
    25%  {background: yellow;}
    50%  {background: blue;}
    100% {background: green;}
    }
```

- 改变背景色和位置

```
    @keyframes myfirst
    {
    0%   {background: red; left:0px; top:0px;}
    25%  {background: yellow; left:200px; top:0px;}
    50%  {background: blue; left:200px; top:200px;}
    75%  {background: green; left:0px; top:200px;}
    100% {background: red; left:0px; top:0px;}
    }
    
    @-moz-keyframes myfirst /* Firefox */
    {
    0%   {background: red; left:0px; top:0px;}
    25%  {background: yellow; left:200px; top:0px;}
    50%  {background: blue; left:200px; top:200px;}
    75%  {background: green; left:0px; top:200px;}
    100% {background: red; left:0px; top:0px;}
    }
    
    @-webkit-keyframes myfirst /* Safari 和 Chrome */
    {
    0%   {background: red; left:0px; top:0px;}
    25%  {background: yellow; left:200px; top:0px;}
    50%  {background: blue; left:200px; top:200px;}
    75%  {background: green; left:0px; top:200px;}
    100% {background: red; left:0px; top:0px;}
    }
    
    @-o-keyframes myfirst /* Opera */
    {
    0%   {background: red; left:0px; top:0px;}
    25%  {background: yellow; left:200px; top:0px;}
    50%  {background: blue; left:200px; top:200px;}
    75%  {background: green; left:0px; top:200px;}
    100% {background: red; left:0px; top:0px;}
    }
```

# CSS3 多列

## CSS3 创建多列

> `column-count` 属性规定元素应该被分隔的列数

- 把 div 元素中的文本分隔为三列

```
    div {
        -moz-column-count:3; 	/* Firefox */
        -webkit-column-count:3; /* Safari 和 Chrome */
        column-count:3;
    }
```

## CSS3 规定列之间的间隔

> `column-gap` 属性规定列之间的间隔

- 规定列之间 40 像素的间隔

```
    div {
        -moz-column-gap:40px;		/* Firefox */
        -webkit-column-gap:40px;	/* Safari 和 Chrome */
        column-gap:40px;
    }
```

## CSS3 列规则

> `column-rule` 属性设置列之间的宽度、样式和颜色规则

- 规定列之间的宽度、样式和颜色规则

```
    div {
        -moz-column-rule:3px outset #ff0000;	/* Firefox */
        -webkit-column-rule:3px outset #ff0000;	/* Safari and Chrome */
        column-rule:3px outset #ff0000;
    }
```

# CSS3 用户界面

## `CSS3 Resizing`

> 在 CSS3，resize 属性规定是否可由用户调整元素尺寸。

- 规定 div 元素可由用户调整大小

```
    div {
        resize:both;
        overflow:auto;
    }
```

## `CSS3 Box Sizing`

> `box-sizing` 属性允许您以确切的方式定义适应某个区域的具体内容。

- 规定两个并排的带边框方框

```
    div {
        box-sizing:border-box;
        -moz-box-sizing:border-box;	/* Firefox */
        -webkit-box-sizing:border-box;	/* Safari */
        width:50%;
        float:left;
    }
```

## `CSS3 Outline Offset`

> outline-offset 属性对轮廓进行偏移，并在超出边框边缘的位置绘制轮廓。

- 规定边框边缘之外 15 像素处的轮廓

```
    div {
        border:2px solid black;
        outline:2px solid red;
        outline-offset:15px;
    }
```


> 没事看看, 虽然不写前端的东西了, 不过多了解总是不会亏的~~~