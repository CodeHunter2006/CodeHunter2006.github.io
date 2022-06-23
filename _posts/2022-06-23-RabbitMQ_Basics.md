---
layout: post
title: "RabbitMQ Basics"
date: 2022-06-23 18:00:00 +0800
tags: MQ
---

![RabbitMQ](/assets/images/2022-06-23-RabbitMQ_Basics_1.webp)
RabbitMQ 是实现了 AMQP(Advanced Message Queuing Protocol) 的开源 MQ。由 Erlang 语言写成。

- 优点：

  - 保证数据不丢失
  - HA
  - 高级功能：
    - 死信队列
    - 消息重试
  - 社区活跃、资料完善、适配各种语言、被中小公司广泛使用

- 缺点：
  - 吞吐率相对 kafka 和 RocketMQ 较低，每秒几万的级别
  - 集群扩展比较麻烦
  - erlang 语言比较小众，难以改造

# 核心结构

![struct](/assets/images/2022-06-23-RabbitMQ_Basics_2.jpg)

- Broker
  物理主机(当然也可以是虚机)

- Virtual Host
  虚拟主机，相当于 namespace，不同 Virtual Host 间的 Exchange 和 Queue 是隔离的。

  - 一个 Broker 中可以有多个 Virtual Host
  - 同一个 Virtual Host 中的 Exchange 和 Queue 不可以重名

- Connnection
  Producer 或 Consumer 与 RabbitMQ 间建立的 TCP 长连接

- Channel
  信道，是在 Connection 基础上建立的虚拟连接，一般的操作其实都是通过 Channel 完成的

  - 可以让不同进程利用各自的 Channel 相互隔离，也可以不同业务逻辑相隔离
  - 多个 Channel 复用了 Connection 的通信能力，避免各进程建立过多 Connection 浪费资源
  - 每个 Channel 都有一个 channel ID

- Queue
  消息队列，相当于 kafka 里的 partition

- Exchange
  交换机，接收 Producer 的消息，然后根据不同的规则将消息分发到不同的 Queue。
  - Binding key
    也叫 Routine key，在 Producer 向 Exchange 发送消息时，必须指定一个 Binding key。
  - 四种 Exchange 类型：
    - direct
      直接匹配。对消息的 Binding key 精确匹配后，转发到对应的 Queue 上。
    - fanout
      扇出广播。不处理 Binding key，直接转发到所有与它绑定的 Queue 上。
    - topic
      主题。根据 Routine key 的通配符匹配情况转发到对应的 Queue 上，可能同时匹配多个 Queue。
      - 匹配逻辑：
        - Routine key 必须是字符串，单词间以`.`分隔
        - `#`可以匹配一个或多个单词
        - `*`可以匹配一个单词
        - 合法的匹配 Routine key: `*.Test`, `Test.#`, `*.Test.*`
    - headers
      不再通过 Binding key 绑定，而是通过 Arguments 绑定。每个 Binding 可以设置多个消息参数，可能同时匹配多个 Queue。

# 安装流程

- `docker run -d -p 15672:15672 -p 5672:5672 rabbitmq`

  - 5672 是 MQ tcp 消息端口
  - 15672 是管理 http 页面端口

- `docker exec -it 容器id /bin/bash`进入容器

- 执行`rabbitmq-plugins enable rabbitmq_management`启动管理页面插件

- 来到`/etc/rabbitmq/conf.d/`目录执行`echo management_agent.disable_metrics_collector = false > management_agent.disable_metrics_collector.conf`
  避免产生权限错误

- 最后执行 `docker restart 容器id` 重启后配置生效，然后就可以在`http://localhost:15672`打开管理页面了

- 测试：
  1. 在 Queues 页面增加一个 TestQueue1
  2. 在 Exchanges 页面增加一个 direct 类型的 TestExchanges1，并设定 Bindings，绑定关键字 "test" 到 TestQueue1
  3. 在 Exchanges 页面 Publish message 发送一条消息，填入 Routing key: "test"
  4. 在 Queues 页面 Get Message，收一条消息

```Go
// Go consumer demo
package main

import (
	"log"
	"math/rand"
	"time"

	"github.com/NeowayLabs/wabbit"
	"github.com/NeowayLabs/wabbit/amqp"
)

func main() {
	log.Printf("server start")
	defer log.Printf("server stop")

	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	bindOpt := wabbit.Option{"autoAck": false}
	msgs, err := ch.Consume(
		"TestQueue1", // queue
		"",           // consumer
		bindOpt,
	)
	failOnError(err, "Failed to register a consumer")

	go func() {
		for d := range msgs {
			log.Printf("Received a message: \"%s\" with tag: %v", d.Body(), d.DeliveryTag())

			time.Sleep(time.Duration(rand.Int31n(10)) * time.Second)
			log.Printf("Process %v success", d.DeliveryTag())

			d.Ack(false)
		}
	}()

	var forever chan struct{}
	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
}

func failOnError(err error, msg string) {
	if err != nil {
		log.Panicf("%s: %s", msg, err)
	}
}
```

- [github.com/streadway/amqp](https://github.com/streadway/amqp) 封装了 RabbitMQ 的 Go 客户端
  - 参考文档：[pkg.go.dev/github.com/streadway/amqp](https://pkg.go.dev/github.com/streadway/amqp)
- 代码中使用了[github.com/NeowayLabs/wabbit](https://github.com/NeowayLabs/wabbit)它是对上面包的封装，提供了 UT 接口

# 其他特性

# 常用命令

# 常见问题
