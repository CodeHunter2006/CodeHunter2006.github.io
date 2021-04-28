---
layout: post
title: "Redis Transaction"
date: 2021-04-30 10:00:00 +0800
tags: Redis
---

![transaction](/assets/images/2021-04-30-Redis_Transaction_1.webp)
Redis 可以通过自带的 Transaction 命令实现乐观锁版的 ACID 事务。

# 涉及命令

`WATCH key [key ...]`
对指定的 key 进行监控，在最终执行前进行检查，如果有 key 发生了变更，则放弃事务返回失败

`UNWATCH`
取消对所有 key 的监控

`MULTI`
标志此客户端连接开启一个事务，本质就是在服务端对客户端 session 进行一个`REDIS_MULTI`标记。
开启事务后，只有`MULTI/EXEC/DISCARD/WATCH/UNWATCH`这 5 个命令会立即执行，其他数据写入、读取命令都会被加入**命令队列**

`DISCARD`
放弃事务，删除已保存的命令队列、取消 watch

`EXEC`
执行事务，无论是否执行都取消 watch。
首先检查 watch 的 key 是否发生变更，如果没有变更，顺序执行任务队列的所有任务，并返回每条命令的返回值组成的集合；
如果 watch 的 key 发生变更，则放弃执行，返回一个空集合；
如果命令执行中有出错命令，则错误结果会记录到返回集合中，继续执行下面的命令

# 基本流程

![transaction](/assets/images/2021-04-30-Redis_Transaction_2.png)

1. `WATCH` 开启监控指定的 key
2. `get xxx` 读取各种信息，通常是上面的 key，计算出事务需要执行的一系列动作
3. `MULTI`开启事务命令入队状态
4. `set xxx` 向命令队列中添加一系列写入和读取命令...。
   添加命令时会检查语法，如果有错误则关闭`REDIS_MULTI`标记并返回错误。
   如果添加命令时语法正确，则加入队列并返回`QUEUED`
5. `EXEC` 执行命令返回结果
6. 如果 5 返回了失败，则根据逻辑，可能重新进入流程 1

# 注意点

- ACID 保证：
  - Atomicity 单线程保证原子性
  - Consistency 由 watch 保证逻辑一致性
  - Isolation 由单线程+watch 保证隔离性
  - Durability 写入的数据是稳定的，由持久化写盘保证
- Redis Transaction 基于类 CAS(Compare And Swap)操作可以实现**乐观锁**，另外还可以用**RedLock**实现悲观锁
- Redis 执行命令时**没有 Rollback**动作，发生问题会继续执行后面的命令。
- **命令入队时**会做语法检查，但是不做逻辑检查。

  - 比如一个 key 是 string 型，执行`INCR`会发生错误；或者用`HSET`执行在 string 类型上。

- Redis Transaction 类似 CAS 机制，是否会出现**ABA**问题？

  - 答：不会。
    Redis 的 Watch 底层实现，是在被 Watch 的 key 增加一个"Watch 链"字段，链上是所有 Watch 了该 key 的 Client Session。
    当 key 被修改时，就会在链上的 ClientSession 结点进行`REDIS_DIRTY_CAS`标记，之后在执行`EXEC`时就会用作判断。所以不存在 ABA 问题。

- 在**Cluster**模式中，由于 key 被分配在不同的结点，在添加命令队列时，如果 key 不在当前连接的结点上，就会报`MOVED`错误并立即返回。

# 其他

- 为什么执行命令队列时发生错误不回滚？
  Redis 是效率为先的，如果考虑回滚会导致整体阻塞、效率低；Redis 做了语法检查，逻辑错误是使用者自己的问题
