---
layout: post
title: "Kafka 丢消息问题"
date: 2021-09-13 22:00:00 +0800
tags: MQ
---

![Basic Process](/assets/images/2021-09-13-MQ_LostMessage_1.png)
记录 Kafka 的消息防丢机制

- 基本流程分三个阶段可能丢消息：

  - 发送阶段
    遇到高延迟，Producer 会多次重发消息，直到 Broker Ack 确认，过程中 Broker 会自动去重。
    如果超时 Producer 产生异常，应用捕获提示。
  - 存储阶段
    Broker 先刷盘再 ack 确认，即便 ack 失败消息不回丢失。如果多次重试直到 Producer 接收，可能会导致(Producer)消息积压。
  - 消费阶段
    Broker 向 Consumer 发数据，一段时间未接收，自动重发，知道 Consumer Ack 确认。Consumer 要注意幂等处理。

- 其他问题点：
  1. 异步刷盘(NSYNC_FLUSH，过早 Ack)，改为同步刷盘
  2. 存储介质可能损坏，建议采用 RAID 10(读作："幺零")或分布式存储
  3. 在 Consumer 端，不要启用自动 Ack，要由业务代码结果决定是否 Ack
  4. 在 Topic 创建时，就划分好合理的 Partition 数量和 Replica 数量，要分布在不同 Broker 上

![RAID 10](/assets/images/2021-09-13-MQ_LostMessage_2.png)

- RAID(Redundant Array of Independent Disks, 独立硬盘冗余阵列) 是一套磁盘冗余存储方式，用于降低磁盘介质损坏带来的丢数据问题。
- RAID 0
  将数据切分后，分段放在两个磁盘存储，这样存储速度快(每个只写一半数据)、丢失数据时也会保留一半。
- RAID 1
  将完整数据写入两个完全相同的磁盘，并行写入，这样其中一个损坏时另一个也会保存数据。
  由于两个盘写入速度不完全相同，相比单盘写入要稍稍慢一点。
- RAID 10
  结合了 RAID 0 和 RAID 1 的特性，由两组磁盘组成阵列，每组内有两个磁盘以 RAID 1 形式结合，组间以 RAID 0 形式结合。
  这样写入耗时相当于单盘的一半，同时具有备份冗余，不过需要 4 个磁盘。
