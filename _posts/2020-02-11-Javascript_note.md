---
layout: post
title: "Javascript语法、常用函数"
date: 2020-02-11 21:00:00 +0800
tags: Javascript
---

## 数组

`array.length = 0`
清空数组元素

`array.splice(index, length, element...)`
将数组中一段元素，替换成另一段元素

- element 参数为空，则不插入元素，相当于删除
- length 参数为 0，为不删除元素，相当于插入

## null undefiend

- null 表示 object 的一个特殊值，表示"空指针"，是关键字
- undefiend 是一个变量，表示该变量不存在
- null 和 undefiend 在做 bool 运算时相当于 false
