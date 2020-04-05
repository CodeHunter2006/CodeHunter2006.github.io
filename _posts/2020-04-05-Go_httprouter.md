---
layout: post
title: "HttpRouter"
date: 2020-04-05 22:00:00 +0800
tags: Go Http Algorithm HighConcurrency
---

[HttpRouter](https://github.com/julienschmidt/httprouter) 是一个用 Go 语言写的开源 url 解析器，对 Go 原生的`net/http`有一定的性能改善。
其实 Router 的性能只要不存在瓶颈，并不会对效率影响太大(路由耗时 10~22ms)。但是可以学习一下数据结构和设计思路。

Router 的任务其实就是快速的从 Url 中抽取路由、参数、Restful 参数等信息，尽快的找到对应的 Handler 来处理。

## 参考数据结构

HttpRouter 之所以速度较快，是因为采用了类 Radix Tree 的数据结构。
常用于路由的简单数据结构有下面两种：

### Trie Tree(字典树)

每个字母作为一个节点，下一个字母作为子节点，组成字典。

Trie Tree 作为路由匹配树有如下缺点：

- 树的深度和路由字符串长度正相关
- 占用较多内存
- 字符串越长匹配越慢，类似链表结构

### Radix Tree(基数树)

每个节点可以为一个子字符串，下一个子字符串作为子节点，组成字典。

Radix Tree 作为路由匹配树有如下缺点：

- Restful 的路由设计中，同一个 url 可能对应多个动作(如 put/post/patch)，无法用一棵树表达。通常用多棵树独立表达，比较浪费内存。

## HttpRouter 中的概念

### node

路由树中的节点

### nType

node type，几种类型：

- static 非根节点的普通字符串节点
- root 根节点
- param(wildcard) 参数节点，如:`id`
- catchAll 通配符节点，在路由只能出现一次，只能在结尾，如: `*anyway`

### path

到达节点时经过的字符串路径

### indices

子节点索引，当子节点为非参数类型(wildChild 属性为 false)，则把子节点首字母放入一个数组，以便快速匹配

### wildChild 节点

如果子节点是参数，则 indices 为空，同时 wildChild 属性为 true

### catchAll

通配符节点

## HttpRouter 规则

static(字符串节点)/param(参数节点)/catchAll(通配符节点)，三者是互斥的。
一个父节点的子节点，只能是`一个param`或`一个catchAll`或`一个或多个static`
