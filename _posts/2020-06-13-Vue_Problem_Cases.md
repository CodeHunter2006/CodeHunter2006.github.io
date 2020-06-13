---
layout: post
title: "Vue 问题案例"
date: 2019-08-18 23:00:00 +0800
tags: Vue Javascript
---

![Vue](/assets/images/2020-06-13-Vue_Problem_Cases_1.jpeg)
总结 Vue 常见问题

# 使用饿了么(Element)组件的页面卡死

- 问题现象
  页面使用大量饿了么组件，在某种条件下，执行某个界面显示的动作时整个页面卡死。
- 分析解决过程
  1. 通过浏览器 log，可以看到当问题发生时，会打印以`ResizeObserver loop limit exceeded`开头的一段 log，之后 log 显示发生了一系列递归渲染
  2. 用渲染检测工具观察，发现不断渲染，导致卡死。而且渲染的区域是一个 Element 的 Table 组件
  3. 对发生问题的函数分析，其中有几个变量与 Table 组件绑定，逐个注释，发现其中一个数组变量会导致发生卡死
  4. 对这个数组变量进一步分析，发现这个变量只有被赋值并清空后，再显示的时候就会发生卡死
  5. 将变量的清空方法从`xxxArray.length = 0;`改为`xxxArray.splice(0, xxxArray.length);`，问题解决
- 问题原因
  - Vue 对象由基本的 Dom 对象和 Vue Wrapper 组成。通常 Javascript 数组用`xxxArray.length = 0;`置空，但是如果`xxxArray`是一个 Vue 对象`.length = 0`只能将 Dom 对象部分置空，而其 Wrapper 部分并没有被置空，并且监听它变化的组件也无法获知变化。在这个数组被填充内容并错误的"置空"后，Dom 对象和 Vue Wrapper 就产生了差异。
  - Element 的 Table 组件显示时，会根据要显示的内容进行宽度计算、排版，这时，由于上面数组长度的不一致性，导致在某个算法中进行了递归循环渲染，进而导致卡死。
- 解决方案
  - 对 Vue 对象操作时，不要自己直接修改 Dom 属性，要利用 Vue 提供的重写过的函数操作。例如对数组置空，可以用`xxxArray.splice(0, xxxArray.length);`，因为 Vue 重写了 Array 的 splice 函数；也可以用`xxxArray = [];`，因为 Vue 重写了 Array 的 set 函数。
  - 某些情况下，可能会写出造成递归循环的错误逻辑代码。例如：A 监听 B、B 监听 C，这时在 A 监听 B 的代码中有修改 C 的代码，就可能造成循环。在写代码时要注意，或分析问题时要考虑这种情况。

PS: 网上有一些文章，给 Element 的 Table 设置 width 属性来避免`ResizeObserver...`错误，这种方法虽然可以避免计算宽度从而解决问题，但是没有抓住实质，并且浪费了组件本身自动调节宽度的功能。
