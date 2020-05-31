---
layout: post
title: "Kafka 基础"
date: 2020-05-31 18:00:00 +0800
tags: MQ
---

![MongoDB](/assets/images/2020-05-31-Kafka_Basic_1.jpg)
Apache Kafka 是一个分布式、开源消息队列(MQ message quere)系统，由 Scala 写成。可为网络服务提供一个高通量、低延迟的异步消息中间件。

- Scala 也是基于 JVM，可以和 Java 混编
- Kafka 在 2011 年初被 LinkedIn 开源，后由 Apache 开源维护

# MQ 的应用场景

MQ 类似 Queue，符合 FIFO(First In First Out)规则，通常用于生产者消费者模式。

- 大并发下的削峰作用
  由于 Kafka 集群的性能高、高可用，所以常用于大并发下将任务暂存，然后异步处理，避免实时系统超负荷导致雪崩。
- 对实时性要求不高的异步处理
  对于对响应速度要求不高的场景，可以利用异步处理架构提高系统的稳定性。比如发邮件系统。
- 调用时机解耦合
  可以利用 Kafka 作为消息中间层进行解耦合。上游只负责生产数据，不用非等下游返回才能继续执行

# 核心结构

![MongoDB](/assets/images/2020-05-31-Kafka_Basic_2.jpeg)

- Topic(主题)
  消息按照 Topic 进行分类，在一个类型中进行生产、消费
- Procucer(生产者)
  向 Topic 发送消息
- Consumer(消费者)
  从 Topick 取消息消费。分为 push 和 pull 两种消费模型
  - push 模式，也叫"发布订阅模式"，Consumer 先订阅指定的 Topic，当有新消息时，Kafka 主动将消息 push 给消费者客户端
  - pull 模式，由 Consumer 主动拉数据。
- Broker(服务)
  Kafka 集群是由多个实例组成的，每个服务实例称为 Broker

![MongoDB](/assets/images/2020-05-31-Kafka_Basic_3.jpeg)

- Partition(分区)
  为了实现扩展性，一个 Topic 可以分不到多个 Broker 上，每个 Broker 上对应这个 Topic 分配一个 Partition。每个 Partition 是一个有序队列，Partition 中的每条消息会分配一个 id(offset)。kafka 只能保证一个 Partition 中的消息顺序发给 Consumer，不保证一个 Topic(多个 Partition 间)的顺序。Kafka 集群会自动分散分配 Partition，一个 Broker 不会存储同一个 Topic 的多个 Partition，创建时会失败，这样可以更好的利用磁盘的吞吐量。
- Partition 副本
  为了实现高可用、避免单个 Broker 宕机后无法使用的情况，一个 Partition 可以有多个副本。各个副本与原 Partition 以及各个副本间不允许在同一个 Broker 上，创建时会失败。往同一个 Topic 发送消息时，默认以轮训的方式发到 Partition，也可以自己指定 Hash 方式确定 Partition。
  - 订阅的最小粒度是 Topic，不可以订阅 Partition
  - Partition 中存储的每条数据包含：index(索引)、log(内容)、timeIndex(时间索引)
- Leader
  原 Partition 所在的 Brokder 也叫 Leader，负责对应 Partition 的读写
- Follwer
  副本 Partition 所在的 Broker 也叫 Follwer，副本在原 Partition 失效后起作用
- Zookeeper
  分布式文件系统，Zookeeper 集群可以为上面各个角色保存 meta data(元数据)，来保证系统可用性。可以通过 Zookeeper 查询到 Kafka 的各种信息。

# 性能和特性

## Topic 有序机制

同一个 Topic 的多个 Partition 间的顺序不能保证，如果想整个 Topic 的顺序保证，那只能建一个分区。
如果想保证特定的一些消息有序，可以利用 Hash 等方法，将这些消息发往同一个 Partition，这样无论 Consumer 如何消费，都能保证这些消息被有序消费。

## ACK 应答机制

Producer 向 Kafka 发送数据后，Kafka 会以下面三种方式应答:

- 0, 生产数据不需要 ACK。速度较快，数据可能丢失(例如 Broker 宕机)
- 1, Leader 写入磁盘后给响应，速度一般，可能数据重复。
- -1, Follwer 都写成功后给响应，速度慢，可能数据重复。

# 常用命令

## 创建 Topic

创建 Topic 时要指定 Zookeeper、Partition 的数量、每个 Partition 的副本数
`./kafka-topics.sh --create --zookeeper node01:2181,node02:2181,node03:2181 --topic test1 --partition 3 --replication-factor 2`

## 启动 Producer

Producer 启动参数要有 Kafka 集群节点
`./kafka-console-producer.sh --broker-list node01:9092,node02:9092,node03:9092 --topic test1`

## 启动 Consumer

Consumer 启动参数需要填 Zookeeper，在 Zookeeper 会自动保存该 Consumer 的消费到的 offset
`./kafka-console-consumer.sh --zookeeper node01:2181,node02:2181,node03:2181 --topic test1`
