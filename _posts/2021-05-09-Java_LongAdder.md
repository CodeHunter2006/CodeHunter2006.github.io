---
layout: post
title: "Java LongAdder"
date: 2021-05-09 20:00:00 +0800
tags: Java Algorithm
---

![Java](/assets/images/2021-05-09-Java_LongAdder_1.png)
jdk8 新增了原子操作类 LongAdder，它对标 AtomicLong，提高了多 CPU 并发下的效率，在这里记录一下。

# 特点

- LongAdder 在**多 CPU、大并发、多写入**情况下性能是 AtomicLong 的上百倍，几乎没有 CPU 资源争抢问题
- 在单 CPU 下，性能比 AtomicLong 还差，不值得使用

# 基本结构

- LongAdder 的最基本的字段是 base，在无争抢的情况下只利用该字段进行 CAS 累加
- 另外通过继承自 Striped64 抽象类，可以利用 cells 数组，数组每一个元素对应于一个 CPU 进行累加
- 最终读取的结果需要累加 base 和 cells 获得

# 算法细节

- AtomicLong 的缺点：多个 CPU 是基于 CAS 轮询累加的，在大并发下争抢严重降低效率

- 一般情况 cells 是空的，只有某 CPU 在 base 字段发生累加抢占时候才在 cells 增加一个元素，通过 hash 进行引用
- cells 元素最大数量为 CPU 数量，这样已经完全没有竞争了，没有必要更多元素
- cells 扩容时，有 CAS 锁
- Cell 类使用`@sun.misc.Contended`注解，避免 CPU 缓存造成伪共享，降低效率
