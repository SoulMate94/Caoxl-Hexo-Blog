---
title: Hexo 黑科技
date: 2018-08-02 15:23:30
categories: Hexo
tags: [Hexo]
---

> 科学表明, 当你{% spoiler 撸管次数过多%}, 会出现看不清字的征兆

<!-- more -->

- {% spoiler 千万别点啊%}
- {% spoiler 叫你别点你还点%}
- {% spoiler 好了, 下面告诉你怎么使用的%}

# 安装/使用

- 安装:

```markdown
    npm install hexo-spoiler --save
```

- 使用:

```
    {% spoiler text %}
```

# 成功展示

- 可以遮蔽正常文本：{% spoiler text %}
- 可以遮蔽带格式的文本：{% spoiler *text* %} {% spoiler **text** %} {% spoiler ~~~text~~~ %}
- 可以遮蔽行内代码，但是效果非常差：{% spoiler `text` %}

{% spoiler 就是这么简单~~ %}
