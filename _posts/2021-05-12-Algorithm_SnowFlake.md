---
layout: post
title: "Algorithm SnowFlake"
date: 2021-05-12 23:00:00 +0800
tags: Algorithm BigData HighConcurrenc
---

![SnowFlake](/assets/images/2021-05-12-Algorithm_SnowFlake_1.jpg)
在分布式系统中希望能够生成一个随时间增长的唯一 ID，这时可以用 Snow Flake(雪花算法)实现。

# 原理

![SnowFlake](/assets/images/2021-05-12-Algorithm_SnowFlake_2.jpg)

- 41 位毫秒时间戳，可以使用 49 年
- 10 位主机 ID，可以用于 1024 台主机
- 12 位同一毫秒内序列号，可以生成 4096 个 ID
- 最前面的 1 位忽略，因为第一位表示负数所以直接取 0

# 优点

- 随时间增长
- 不同主机不会重复，支持分布式部署
- 生成简单，只需要取时间戳和位运算

# 缺点

- 严重依赖服务器时钟，一旦时钟回拨则会导致 ID 重复

# 思考

- 如果不需要随时间增加，用 UUID 就可以
