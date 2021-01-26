---
layout: post
title: "Go Runtime"
date: 2021-01-22 22:00:00 +0800
tags: Go
---

![Go Runtime](/assets/images/2021-01-22-Go_Runtime_1.jpg)
记录一些 Go Runtime 的设计点

# 调度

# 内存

# GC

# 优化经验

- 协程池的重要性远没有 Java,CPP 中线程池那么重要
  - 协程的生成不涉及系统调用，需要的栈资源也很少
  - P 和全局都做了 dead G 的缓存
  - 如果协程池实现的不好，反而因为锁影响了性能
  - 对于并发控制、保护资源，完全可以选择其他方式
- 什么时候需要协程池？

  - 主要用于减少栈扩容和缩容，有些场景下栈扩容和缩容消耗较多 CPU(可用 pprof 查看 morestack)，比如长连接
  - 大量维持连接的协程可以不用扩容栈，复杂任务交给任务携程处理，此类携程的数量比较少

- 利用 sync.Pool
  对于频繁分配的堆上对象，可以使用 sync.Pool，减少分配频次，进而降低 GC 频率

- 一点点拷贝胜过指针
  如果对象较小，可以考虑用栈上对象+传参的方式实现，这样可以减少 GC 对 CPU 消耗

- slice 和 map 如果提前知道容量范围，可以初始化一个合适的容量，减少扩容

- 用 json-iterator 替换 encoding/json 减少反射的使用，提高效率

- 方便的 pprof 工具
  - https://github.com/google/gops
  - 可以很方便的内嵌入服务中，并可以在线查看 pprof 信息
