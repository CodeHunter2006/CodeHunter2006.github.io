---
layout: post
title: "分布式事务(最终一致性)"
date: 2021-06-19 23:00:00 +0800
tags: Algorithm Distribute
---

![XA](/assets/images/2021-06-19-Distributed_Transaction_1.jpeg)

# 概念

事务是符合 ACID 特定的一系列操作，分布式事务是在分布式环境下各结点逻辑实现最终一致。

- 难点：

  - 数据分布在不同的独立主机上的数据库(如用户、账户、交易三个独立的主机)，但是需要实现事务操作，而网络、主机都可能出问题
  - 所以需要一个**独立的协调者**进行事务管理操作，协调各个**参与者**

- 分布式事务和分布式一致性协议的不同
  - 分布式强一致性协议(如 Raft)中 Follower 只是 Leader 的备份，各个 Follower 存储的数据是相同的。
    而分布式事务中各个参与者存储的数据是不同的，需要配合完成一个事务。
  - 分布式一致性协议要求只要达到 quorumNumber 就算确认完成，而分布式事务必须所有参与者都返回 Commit OK 才可以。
  - 在分布式事务中，参与者可以返回中止，即可以影响事务的执行过程；而 Raft 中，Follower 只是投票和记录，不参与流程决策。

# 2PC(Prepare Commit)

- 在单个主机的数据库可以实现事务，但是如果数据库分布在多台独立主机，那就需要下面流程：

- 基本流程：(假设三台主机分别负责用户、账户、交易)

  - 客户端发起一个事务请求
  - 事务协调者向三台主机同时发出 Prepare
  - 三台主机收到 Prepare 后，各自在数据库开启事务并准备好操作(redo log)，然后向协调者返回 OK
  - 事务协调者收到三台主机的 Prepare OK 后，发出 Commit
  - 三台主机收到 Commit 后，向协调者返回 OK
  - 事务协调者收到三台主机的 Commit OK 后，向客户端确认完成本次事务

- Prepare 出错流程：

  - 在 Prepare 阶段，一台主机返回 Error，或者超时没有返回
  - 事务协调者向三台主机发出 Rollback
  - 两台主机收到 Rollback 后回复 Rollback OK
  - 事务协调者收到 Rollback OK 后向客户端确认本次事务失败

- Commit 阶段重试：

  - 在 Commit 阶段，一台主机没有响应
  - 事务协调者收到另外两台的 Commit OK，不断重试 Commit
  - 直到成功后，事务协调者向客户端确认本次事务成功

- 2PC 的缺点：
  - 参与者的本地事务在 Prepare 阶段锁定资源，如果有其他事务要修改相同资源，会造成**同步阻塞**、性能下降
  - **协调者单点故障**，一旦协调者出问题则系统失效
  - Commit 阶段，如果其中一个参与者没能收到 Commit 消息，则系统会出现(Commit 后的)**数据不一致**

# 3PC

为了解决 2PC 的缺点，引入了三阶段提交

- 将 2PC 的 Prepare 拆分成 canCommit 和 preCommit 两个阶段
- canCommit 与 2PC 的 Prepare 类似
- preCommit 阶段，参与者记录 redolog 后立即返回 OK，但是并没有执行，而是开启一个定时任务
- 协调者发送 Commit 请求，各参与者收到后执行 Commit
- 如果协调者发送的 Commit 有参与者未收到，则根据定时任务，超时后自动执行 Commit

- 3PC 的改进点：

  - 如果进入第三阶段 Commit，不论协调者发生故障还是网络异常，参与者都能执行 Commit，避免数据不一致

- 3PC 的缺点：

  - 如果协调者第三阶段发出 Rollback 请求，而有参与者没有收到，超时后执行了 Commit，则仍然会有数据不一致

- 根据业务进行数据补偿/修正，可以弥补 3PC 的缺点
  - **MQ 事务**
    利用消息中间件来异步完成事务的后一半更新，实现系统的最终一致性。这个方式避免了像 XA 协议那样的性能问题。
  - **TCC 事务**
    TCC 事务是 Try、Commit、Cancel 三种指令的缩写，其逻辑模式类似于 XA 两阶段提交，但是实现方式是在代码层面来人为实现。

# XA 协议

XA 协议是由 X/Open 组织提出的分布式事务处理规范，主要定义了事务管理器(TM)和局部资源管理器(RM)之间的接口。
目前主要的数据库，比如 Oricle、DB2、MySQL5.0InnoDB 都支持 XA 协议。

## XA 语法

- 按步骤：

  - `XA {START|BEGIN} xid [JOIN|RESUME]`
    三阶段的第一阶段：开启 xa 事务，这里 xid 为全局事务 id
    - `XA END xid [SUSPEND [FOR MIGRATE]]`
      结束事务
  - `XA PREPARE xid`
    三阶段的第二阶段，即 Prepare
  - `XA COMMIT xid [ONE PHASE]`
    `XA ROLLBACK xid`
    三阶段的第三阶段，即 Commit/Rollback
  - `XA RECOVER XA RECOVER [CONVERT xid]`
    查看处于 Prepare 阶段的所有事务

## 关于 Seata

![Seata](/assets/images/2021-06-19-Distributed_Transaction_2.png)
Seata 是阿里推出的一款开源分布式事务解决方案，目前有 AT、TCC、SAGA、XA 四种模式。
