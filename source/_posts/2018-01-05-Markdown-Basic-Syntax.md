---
title: Markdown 基础语法
date: 2018-01-05 16:26:39
categories: DevTool
tags: [Markdown, Hexo]
---

# Markdown 语法

> Markdown 常用与书写博客,开发文档等

> '.md' 和 '.markdown' 都是被普遍支持的扩展名，不过 '.md' 更加简单和方便。

<!--more-->

# 常用语法

## 标题
```
    # 表示一级标题
    ## 表示二级标题
```

### Setext 形式

```markdown
    H1
    ====

    H2
    ----
```


> `=` 和 `-` 的数量是没有限制的。通常的做法是使其和标题文本的长度相同，这样看起来比较舒服。或者可以像我一样，用四个 - 或 =。
Setext 形式只支持 `h1` 和 `h2` 两种标题。

## 段落
段落以自然回车作为标记

段落以自然回车作为标记


## 引用
```
    写法: 
        >
```
> 这里就是采用了引用

```
    写法二: 
        > >
```
>> 这里是层次感的引用

```
    写法三: 
        > -
```
> - 引用里面有序显示

```
    写法四: 
     > 树 
     > > 二叉树
     > > > 多重树 
```
 > 树 
 > > 二叉树
 > > > 多重树 

## 斜体,加粗
```
    斜体写法：    *内容*
    粗体写法：    **内容**
    又粗又斜：    ***内容***
```
*斜体*
**粗体**
***又斜又粗***

## 有序/无序
```
    * + - 均可以
    需要有层次感 缩进即可
    如:
    * 第一层
      + 第二层
        - 第三层
```
* 第一层
  + 第二层
    - 第三层


## 超链接

### 普通链接

```
    写法: [显示的文字](链接地址)
```

- [曹贤亮的博客](caoxl.com)

### 指向本地的链接

```
    [icon.png](./images/icon.png)
```

### 包含 `'title'` 的链接:

```
    [Google](http://www.google.com/ "Google")
```

[Google](http://www.google.com/ "Google")

> title 使用 ' 或 " 都是可以的。

## 图片

```
    写法: ![](图片地址)
```

![我的博客LOGO](http://blog.caoxl.com/css/images/logo.png)

### 指定图片的显示大小

```
    <img src="http://blog.caoxl.com/css/images/logo.png" alt="GitHub" title="GitHub,Social Coding" width="50" height="50" />
```

<img src="http://blog.caoxl.com/css/images/logo.png" alt="我的博客LOGO" title="我的博客LOGO,Social Coding" width="50" height="50" />

## 分割线

```
    写法: ___ 或者 ---
```

 - 分割线
 ___
 
 ---


## 表格
> 使用- |符号把内容分割为你认为合适的表格样式就好。
  使用:符号标识对齐。
  
```
    表头1|表头2|表头3
    :----|:-----:|-----:
    左对齐|居中对齐|右对齐
```

表头1|表头2|表头3
:----|:-----:|-----:
左对齐|居中对齐|右对齐


## 保存
> 最后将markdown编写的文档存为 .md 格式，就可以用对应的工具查看效果和编辑了。



# 扩展

## 下划线

```
    <u>Underlined Text</u>
```
- 效果 :
<u>为文字增加下划线</u>


## 删除线

```
    ~~要划除的行内内容~~
```

- 效果 :
~~要划除的行内内容~~


## 标注重点

```
    `重点`
```

- 效果 :
`这里是重点`

## 兼容 HTML

```html
    <table>
      <tr>
        <td>1</td>
        <td>2</td>
      </tr>
      <tr>
        <td>3</td>
        <td>4</td>
      </tr>
    </table>
```

<table>
  <tr>
    <td>1</td>
    <td>2</td>
  </tr>
  <tr>
    <td>3</td>
    <td>4</td>
  </tr>
</table>

# 支持Markdown语法的工具

 - 有道云笔记
 - Showdoc
 