---
layout: post
title: "markdown基础"
tags: markdown
---

*markdown*的原理是将纯文本转换为 html，从而获得更好的视觉效果。
但这里的“纯文本”本身就是有一定格式的，看起来就是很舒服的，经过转换效果更佳。  
Github 中的 markdown 添加一些新语法，叫作 GFM(Github flavored markdown)
[参考网址](http://daringfireball.net/projects/markdown)

# 常用样式: (可在源码中查看)

<br/>换<br/>行<br/>

_斜体_

**粗体**

1. 顺序
2. 列表

- 无序
- 列表

## 一级标题

# 二级标题

### 三级标题

###### 六级标题

横线

---

外部图片：
![picture_alt](https://ss0.bdstatic.com/5aV1bjqh_Q23odCf/static/superman/img/logo_top_86d58ae1.png)

本地图片：
![picture_alt](/assets/local_test_pic.png)

- 可以给图片加尺寸

```
<img src="/assets/local_test_pic.png" alt="drawing" width="100" height="28"/>
```

<img src="/assets/local_test_pic.png" alt="drawing" width="100" height="28"/>
* 也可以只加宽度
```
<img src="/assets/local_test_pic.png" alt="drawing" width="100"/>
````
<img src="/assets/local_test_pic.png" alt="drawing" width="100"/>

| 表格标题 1   | 表格标题 2   |
| ------------ | ------------ |
| 表格项目 1.1 | 表格项目 1.2 |
| 表格项目 2.1 | 表格项目 2.2 |

```C++
/*
	代码段，可以设定语言类型进行语法高亮，有些生成器不支持
*/
```

{% raw %}
由于 jekyll 会用到双层花括号"{{xxx}}"进行取值运算，如果要显示双层花括号，要用到一个开关。
{% endraw %}
