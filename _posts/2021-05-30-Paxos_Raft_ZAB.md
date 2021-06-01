---
layout: post
title: "分布式强一致性共识算法 Paxos Raft ZAB"
date: 2021-05-30 23:00:00 +0800
tags: Go
---

# 基本概念

根据**CAP Theorem**，分布式系统不能同时满足三点：Consistency(一致性)、Availability(可用性)、Partition Tolerance(分区容错性)

- CP
  MongoDB、HBase、Redis
- AP
  CouchDB、Cassandra、DynamoDB、Riak
- CA
  RDBMS

- 什么是**一致性**？分为两种：

  - **弱一致性**
    - **最终一致性**
      - DNS(Doman Name System)
      - Gossip (P2P 基础协议，Cassandra 的通信协议)
  - **强一致性**
    - 同步
    - Paxos
    - Raft(multi-paxos)
    - ZAB(multi-paxos)

- 最终一致性示例：
  DNS 修改后，不能立即读到，但是经过一系列结点间同步后，最终可以读到正确的值。

- 分布式系统的**fault tolorence**的一般解决方案是**state machine replication(状态机复制)**，
  通过记录带序列号的状态变更 log 的方式同步各个结点的数据。
- Paxos/Raft/ZAB 是目前业内公认的 state machine replication 解决方案的**consensus(共识算法)**
- 共识算法并不能保证"最终一致性"，还需要 client 行为配合才能实现

# 强一致性共识算法的演化

## 主从同步

- 结构：
  - 一主多从
  - 主接收写请求
  - 主把变动复制到从
  - 主**阻塞等待**所有从确认后才继续
- 缺点：
  - 任意一个从结点失败会导致主被阻塞，虽然保证了一致性，可用性却大大降低。

## Quorum(多数派)

- 基本思想：
  - 每次写入保证`>N/2`个结点写成功
  - 每次读取保证从`>N/2`个结点读出
- 缺点：
  - 在并发情况下，由于任意结点都可写入，可能发生不同结点保存的数据乱序

## Basic Paxos

Paxos 算法的发明者是 Lesile Lamport(他也是 Latex 的发明者)。
他同时提出了`Basic Paxos`、`Multi Paxos`、`Fast Paxos`三种方案，
同时用一个虚拟的名叫 Paxos 的希腊城邦的议会民主制度作为示例。

- 角色介绍：

  - Client
    系统外部角色，请求发起者。比如"民众"。
  - Prosper
    接受 Client 请求，向集群提出提议(propose)。在冲突发生时，起到调节冲突的作用。像"议员"，替民众提出提案。
  - Acceptor(Voter)
    提议接收者，负责投票。只有在投票人数达到 Quorum(半数以上)时，提议才会最终被接受。像"国会投票人员"
  - Learner
    提议接受者，负责 backup，对集群一致性有影响。像"记录员"

- Basic Paxos 两阶段、四步骤：

  - Phase 1a: Prepare
    Prosper 提出一个提案，编号为 N，N 大于这个 Prosper 之前提出的提案编号。请求 Acceptors 的 Quorum 接受。
    此时只是一个提案编号，没有内容。
  - Phase 1b: Promise
    如果 N 大于此 Acceptor 之前接受的任何提案编号，则接受，否则拒绝。
  - Phase 2a: Accept
    如果 Prosper 得知达到了多数派，会发出 accept 请求，此请求包含提案编号 N 以及提案内容。
  - Phase 2b: Accepted
    如果此 Acceptor 在此期间没有收到任何编号大于 N 的提案，则接受此提案内容，否则忽略。

- 通常场景：

  - 按照上面执行完毕后，Prosper 会向 Leaner 发送记录请求，Leaner 记录后返回 response。

- 少数 Accepter 失败场景：

  - 由于仍然可以达到 Quorum，不影响整个流程。

- Proposer 失败场景：

  - 假设在 Proposer 发起 Prepare 后崩溃，那么编号 N 将不会发出 accept 请求，所以编号 N 不会最终接受。
  - 然后另一个 Proposer 可以受理 Client 的请求，重新发起编号为 N+1 的提案，然后正常执行

- 潜在问题：**活锁**(liveness/dueling)
  如果有两个 Proposer，在对方提交新的编号后，自己就提出更大的编号，
  导致整个过程始终卡在 Phase1，就形成了活锁
- 解决方案：
  如果 Prosper 意识到了活锁，则分别执行 random timeout，其中某方会先发出新的编号，执行完毕后另一方才提新的编号即可。

- Basic Paxos 的问题：
  - 实现困难
  - 效率低(2 轮通信确认)
  - 活锁

## Multi Paxos
