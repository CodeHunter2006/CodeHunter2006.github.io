---
layout: post
title: "Zookeeper 分布式锁"
date: 2021-03-23 23:00:00 +0800
tags: Algorithm HighConcurrency Server Zookeeper
---

# 实现思路

- 利用`create`进行独占抢锁，抢到锁的会返回成功
- 利用临时节点保证会话断开(抢到锁的进程崩溃)后能够自动删除节点
- 利用监听机制实现锁释放后可以快速抢到锁
- 利用临时顺序节点，实现**公平锁**，每个节点只要监听前面一个节点就可以
- 只有排在最前面的节点才可以抢到锁
- 如果中间有节点放弃了锁，则后面的自动向前监听

- Zookeeper 实现的是**公平锁**，有"先来后到"，实际上是实现了分布式网络队列

- 与 Redis 的 redlock 相比，Zookeeper 的并发性能差一些(硬盘写入、分布式节点确认)，但是稳定性更好