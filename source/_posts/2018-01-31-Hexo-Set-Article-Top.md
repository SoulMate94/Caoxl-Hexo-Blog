---
title: Hexo 文章置顶功能
date: 2018-01-31 15:26:17
categories: Hexo
tags: Hexo
---

使用Hexo博客中,想使用文章置顶功能,所以倒腾了下


<!--more-->

# 期望

在Hexo Hiker 主题下让某些文章置顶


# 原理

在Hexo生成首页HTML时，将top值高的文章排在前面，达到置顶功能。

修改Hexo文件夹下的 `node_modules/hexo-generator-index/lib/generator.js`，在生成文章之前进行文章top值排序。


# 实操

需添加的代码：

```
    posts.data = posts.data.sort(function(a, b) {
        if(a.top && b.top) { // 两篇文章top都有定义
            if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
            else return b.top - a.top; // 否则按照top值降序排
        }
        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
            return -1;
        }
        else if(!a.top && b.top) {
            return 1;
        }
        else return b.date - a.date; // 都没定义按照文章日期降序排
    });
```

其中涉及Javascript的比较函数：

```
    cmp(var a, var b) {
        return  a - b; // 升序，降序的话就 b - a
    }
```

修改完成后，只需要在`front-matter`中设置需要置顶文章的`top`值，将会根据`top`值大小来选择置顶顺序`top`值越大越靠前。需要注意的是，这个文件不是主题的一部分，也不是Git管理的，备份的时候比较容易忽略。

如:
```
    ---
    title: PHP 必备
    date: 2018-01-31 15:03:04
    categories: PHP
    tags: PHP
    top: 9999
    ---
```

> 上面的废话都可以不用看,直接修改下面的内容即可

以下是最终的 `generator.js` 内容

```
    'use strict';
    
    var pagination = require('hexo-pagination');
    
    module.exports = function(locals){
      var config = this.config;
      // 修改部分-start
      //var posts = locals.posts.sort(config.index_generator.order_by);
      var posts = locals.posts;
        posts.data = posts.data.sort(function(a, b) {
            if(a.top && b.top) {
                if(a.top == b.top) return b.date - a.date;
                else return b.top - a.top;
            }
            else if(a.top && !b.top) {
                return -1;
            }
            else if(!a.top && b.top) {
                return 1;
            }
            else return b.date - a.date;
        });
      // 修改部分-end
      var paginationDir = config.pagination_dir || 'page';
      
      // 修改部分-start
      // var path = config.index_generator.path || '';
      return pagination('', posts, {
      // 修改部分-end
      
        perPage: config.index_generator.per_page,
        layout: ['index', 'archive'],
        format: paginationDir + '/%d/',
        data: {
          __index: true
        }
      });
    };
```