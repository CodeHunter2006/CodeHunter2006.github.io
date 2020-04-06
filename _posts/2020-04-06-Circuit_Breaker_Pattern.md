---
layout: post
title: "架构模式——Circuit Breaker Pattern(断路器模式)"
date: 2020-04-06 12:00:00 +0800
tags: DesignPattern Server
---

![Circuit Breaker](/assets/images/2020-04-06-Circuit_Breaker_Pattern_1.jpeg)
Circuit Breaker(断路器模式)是网络服务端常用的架构模式，提供"熔断机制"、避免"雪崩效应"使系统出问题后有机会恢复正常。

# 问题

- "雪崩效应"
  在网络服务系统中，如果多个服务之间存在依赖关系，并且是生产者、消费者关系，在高并发下，消费者一旦发生临时错误、导致积压，就可能引起上游生产者积压、失效，最中引发整个系统不可用。

在发生雪崩效应时，往往是流量高峰期，大量客户端在系统不可用时，仍然不断重试，导致系统某些服务刚一恢复就又挂掉，迟迟无法恢复。

# 方案

![Circuit Breaker Pattern](/assets/images/2020-04-06-Circuit_Breaker_Pattern_2.jpeg)

# 相关模式

- Retry Pattern
