---
layout: post
title:  "高并发下的拥塞控制方法"
date:   2019-04-03 10:30:00 +0800
tags: HighConcurrency
---

连接限流、服务降级


# 限流算法

计数器，单个IP每分钟60次访问。

漏桶算法，每秒设定可服务个数，超过就不能访问。

令牌桶算法，规定每秒的令牌数，如果一时超过，下面再来的要等待更长时间。

# 服务降级
超时无法获得令牌，可以进入服务降级逻辑，例如提示用户。
上面这些只能单机限流，分布式就无法限流了。

分布式限流、接入层限流。

# 用Redis实现分布式限流

利用lua脚本实现，key对应的value存储连接数，设定超时时间比如2秒。

每次申请权限时判断value是否已经达到最大值(比如500)并增1，同时延后超时时间。实现的效果是2秒内达到500后就会等待两秒才能再申请。

Redis这种分布式限流，适合系统内部做业务层面的限流。

对外的限流要通过接入层限流，一般是用Nginx限流。

