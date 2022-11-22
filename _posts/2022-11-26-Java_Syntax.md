---
layout: post
title: "Java 语法点"
date: 2022-11-26 22:00:00 +0800
tags: Java
---

![Java](/assets/images/2022-11-26-Java_Syntax_1.webp)
记录 Java 与 C++/Go 的不同语法点

## import

- 关于**别名**
  Java 不支持在`import`时定义包或类的别名，如果有同名类需要同时使用，可以直接用包含全包名的类名。由于 Java 的包名就是域名的反向文本，所以不会出现全包名的冲突。
