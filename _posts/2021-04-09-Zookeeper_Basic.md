---
layout: post
title: "Zookeeper 分布设计方案"
date: 2021-04-09 22:00:00 +0800
tags: Algorithm HighConcurrency Server Zookeeper
---

Zookeeper 是一个分布式网络文件系统

- Zookeeper 是主要用于解决一系列分布式协调问题：
  - 分布式锁
  - 分布式 ID
  - 分布式配置
  - 分布式服务注册发现
  - 分布式 HA(High Availability 高可用)
