---
layout: post
title: "jekyll 使用笔记"
date: 2016-01-18 09:00:00 +0800
tags: jekyll
---

# 常用命令

## 生成文件并启动服务

```
bundle exe jekyll serve -I -H 0.0.0.0 -P 8000
```

- `-I` 表示增量编译，只有文件发生变化时才编译相关文件，可以避免无效的全部编译。
- `-H` 设定服务 IP 地址
- `-P` 设定端口

## 只生成文章及相关文件

```
bundle exe jekyll build
```

## 清除生成的文件

```
bundle exe jekyll clean
```

**注意：** 某些情况下，会发生某些未知错误，导致无法正常生成文章(如图片无法更新)。这时，要关闭增量编译,执行 clean，重新生成一遍文件，可能就会修复。

# 关于换行符

现在通常有两种换行符：
`CR` Carriage Return，回车符，"\r"，是以前打字机用来移动到下一行第一个位置的按键。
`LF` Linefeed，ASCII 换行符，"\n"。

- Windows 采用的 CRLF
- Linux/Mac 采用的 LF

在 jekyll 中要用 LF("\n")，否则所有内容将被认为是在同一行的。

# 关于花括号转义问题

{% raw %}
由于 jekyll 会用到双层花括号"{{xxx}}"进行取值运算，如果要显示双层花括号，要用到一个开关。具体字符无法显示，请参照这篇文章源码。
{% endraw %}
