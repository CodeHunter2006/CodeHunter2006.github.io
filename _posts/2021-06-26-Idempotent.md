---
layout: post
title: "幂等问题的应对方案"
date: 2021-06-26 23:00:00 +0800
tags: Server
---

![Idempotnent](/assets/images/2021-06-26-Idempotent_1.png)

# 什么是幂等

对于**写入**的请求，多次执行不会造成**数据不一致**问题，那么这个处理过程具有**幂等性**。

- 幂等接口要求对同一事务重复执行不会造成重复等数据不一致问题
- 幂等接口的返回值可以随操作次数不同而不同，不会影响最终存储

- 非幂等造成**数据不一致**问题的场景：
  - 网络请求下，由于超时未响应，又重复请求了一次，而实际上被执行了两次
  - 页面上的按钮，不小心点击了多次
  - MQ 中间件，重复消费消息
  - 第三方回调接口(如支付成功)，由于网络问题重复回调
  - 其他只要是**异步/网络**操作，都有可能发生幂等问题

# 幂等场景

- 下面 SQL CRUD 是否幂等？

  - **查询** `select * from user where xxx`
    查询不会造成数据不一致，具备幂等性
  - **新增** `insert into user(userid,name) values(1,'a')`
    - 如 userid 为**唯一主键**，即重复操作上面的业务，只会插入一条用户数据，具备幂等性
    - 如 userid**不是主键**，可以重复，那上面业务多次操作，数据都会新增多条，不具备幂等性
  - **修改**
    - **直接赋值** `update user set point = 20 where userid=1`
      直接赋最终值，point 始终一样，具备幂等性
    - **计算赋值** `update user set point = point + 20 where userid=1`
      每次操作 point 数据都不一样，不具备幂等性
  - **删除** `delete from user where userid=1`
    多次操作，结果一样，具备幂等性

- 上面场景中**新增没有唯一约束数据**和**计算赋值**都不具备幂等性，需要有应对方案

- 上面描述的幂等性其实是**短时幂等性**，即如果时间跨度较大，则允许发生**ABA 问题**。
  短时幂等性主要针对网络重试问题；如果要长时间范围内仍然幂等，则每个操作要有唯一 ID。

# Token 机制

- 概要流程：

  1. 客户端请求显示表单
  2. 服务端生成一个 token，记录在 redis，返回给客户端的表单中带这个 token
  3. 客户端填写表单，发送提交到服务端
  4. 服务端检查 redis 中的 token 是否被使用过，如果没用过，则正常处理
  5. 如果服务端检查 token 被使用过，则返回错误，表示不能重复处理，实现幂等

- 上面流程中，服务端何时标记 redis 中的 token "已使用"状态，是一个设计难点

  - 问题：
    - 如果先做标记，再处理，这时如果服务端挂了，那么再没有机会处理
    - 如果先处理再标记，如果处理完未标记就挂了，则之后可能重复处理
  - 方案：
  - 这是一个典型的**数据库和缓存 redis 数据不一致** 问题，
    可以按**先标记再处理**来处理，这样如果未能处理，客户端只要再请求新 token 重试一次即可，
    不会造成重复处理的严重问题。

- token 机制缺点

  - 客户端每次请求，都会额外获取、判断 token 是否存在，为了避免小概率的重试问题，让所有请求过程都发生了额外请求(获取一次 token)。

- token 往往是用 UUID 而不是自增 ID，这样无法从客户端推算出 ID，更加安全。

# 乐观锁机制

乐观锁可以解决"计算赋值型"问题，要求客户端请求时要带着版本号，这样就保证了幂等。当然，这个版本号必须是之前就以某种方式获得过的。
`update user set point = point + 20, version = version + 1 where userid=1 and version=1`

- 乐观锁机制缺点
  - 在进行业务请求前，需要先查询出当前的 version 版本，之后就可以每个请求 version 增 1 了

# UUID 机制(有一致性缺陷)

可以利用 UUID 特性，对于发生问题后严重程度可控的场景，由客户端发出请求时生成一个 UUID，然后在服务端对这个 UUID 进行重复性检测和动作执行。

- 优点：
  - 客户端无需从服务端取得 Token 或 Version，减少一次通信
- 缺点：
  - 有较低的碰撞概率，不能用于要求完全幂等的情况

# 唯一主键机制

这个机制利用了**数据库主键唯一约束**的特性，解决了**insert 场景**下的幂等问题。

在分布式系统中，可以利用**雪花算法**生成**全局唯一主键**，然后在请求中使用。
如果是分库分表的场景下，同一个请求一定要落到同一个数据库和同一个表中，否则主键约束就不起作用了。

- 缺点：
  在数据库存储时，**不能使用自增主键**，所以这个方案对业务可能有影响。
  另外如果客户端生成全局 ID，要利用服务端时间。

# 去重表机制

需要一个"去重表"，表中只有一个字段用于实现数据库的主键唯一约束。流程如下：

![Idempotnent](/assets/images/2021-06-26-Idempotent_2.png)

如上图，业务表和去重表的操作是在同一事务中执行的，这样**利用去重表保证了数据的唯一性同时还避免了去重逻辑污染业务表**。
这个方案要求去重表和业务表必须在同一库中，这样才可以进行事务操作。

这个方案是比较常用的，只要规划好全局唯一主键即可。

# 重复 ID 检测机制优化

用 MySQL 数据库检测 ID 是否重复可以实现，但是在数据量大(单表超过 1 亿)+并发大(单机 1000QPS 以上)的情况下无法满足性能要求，
可以利用两个算法实现在内存检测一轮 ID，如果是否**一定不存在**，以提高检测效率。如果 ID 一定不存在/未执行，则可以执行。

- 用 Bloom Filter 检测，如果检测到**可能存在**，则再到数据库查询
- 用 HyperLogLog 检测，同上

- 在检测时，可能执行事物操作，可以先置内存再操作，这样即使操作失败，未来内存检查只是第一步，还可能再执行

# 无侵入式的幂等 Restful 框架

![Idempotnent](/assets/images/2021-06-26-Idempotent_3.png)

- 客户端发出的每个请求要生成一个 UUID 类型的 RequestID，为了避免全局发生冲突可以加入 UserID
- 在应用网关，用 nginx+lua+redis 的方式检查 RequestID 是否存在，如果不存在则记录到 Redis 并向后调用服务；如果已存在则返回错误
- Redis 中的数据：key:RequestID, value:state(PROC/OK), TTL: 设定一个合理的时间，以保证一定时间内幂等，避免内存撑爆
- 处理服务完成后，要更新 Redis 的状态为 OK，以便幂等判断时返回合理的 ErrorCode
- 客户端需要处理新增的两种 ErrorCode: 201 重复，处理中；202 重复，已处理完毕；

- 使用上述方案实现后，只需要客户端有少量侵入即可实现，服务端只需要通用逻辑处理一下状态设置，业务代码完全无侵入

- 如果没有 nginx 或类似网关，也可以在自己的服务的通用代码中实现相同逻辑，关键点是在 Redis 保存着幂等信息

# 总结

在实现幂等的方案中，尽量不要让系统变得复杂，所以推荐**唯一主键/乐观锁/无侵入 Restful**方式，实现也比较简单。

参考链接：[何为幂等？如何设计？](https://www.toutiao.com/i6709680834904850958/?timestamp=1624511722&app=news_article_lite&use_new_style=1&req_id=20210624131522010135168228123853B0&share_token=08c0877d-4314-468b-90bf-5b8e78382b00&group_id=6709680834904850958&wid=1624511752402)
