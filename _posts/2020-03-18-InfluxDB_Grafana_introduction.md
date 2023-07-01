---
layout: post
title: "InfluxDB + Grafana + Telegraf"
date: 2020-03-18 22:00:00 +0800
tags: Monitor
---

![InfluxDB + Grafana](/assets/images/2020-03-18-InfluxDB_Grafana_introduction_1.jpg)
InfluxDB 是 InfluxData 公司用 Go 语言开发的开源时序型数据库(TSDB Time Series and Spatial-Temporal Database)，利用 Telegraf 代理将数据写入后，结合 Grafana 可以高性能的实现网络实时数据监控。

# 网络监控场景

- 微服务，大量不同种类、数量的服务部署在分布的节点上，产生的监控数据是分散的、结构不同的，需要集中起来观察、监控
- 实时性，服务产生的实时信息汇总起来，实现基本的数据可视化，便于用人眼观察趋势、也便于监控程序报警
- 高并发，采集的信息可能在短时间内产生大的并发，并且我们不希望采集的过程影响服务的并发能力
- 大容量，由于服务数量多、采集点多，会产生大量的数据，上百个服务实例大并发下每秒可能产生 GB 级的监控数据
- 重复性，虽然数量较多，同一个服务的数据格式比较固定、值也总在一定范围内，只是数量、频率较大，正好可以利用
- 暂时性，与实时性相对的，监控数据的价值随时间降低

# InfluxDB 特性

- 分布式和水平伸缩扩展
- 持续高并发写入、无更新
- 数据压缩存储
- 支持任意结构的事件数据
- 提供时间相关聚合函数，可以实时对大量数据进行计算
- 类 SQL 查询语法
- 低查询延时

# InfluxDB Mysql 核心概念对比

| InfluxDB           | Mysql    | 说明                   |
| ------------------ | -------- | ---------------------- |
| database           | database | -                      |
| measurement        | table    | -                      |
| point              | row      | -                      |
| field              | column   | field 支持多种类型     |
| tag                | index    | tag 只支持字符串类型   |
| time               | 自增 id  | 原生基础索引           |
| retention policy   | -        | 数据保存策略, 超时清除 |
| continuous queries | -        | 定时任务               |

# Telegraf

Telegraf 是 InfluxData 公司用 Go 编写的开源代理程序，可收集系统和服务的统计数据，并写入到 InfluxDB 数据库。内存占用小，通过插件系统可轻松添加支持其他服务的扩展。

- 如果个别数据的丢失不重要，可以用 UDP 模式发送数据，减轻网络压力

# Grafana

Grafana 是一个纯 HTML+JS 的 web 应用，是一个开源仪表盘工具，访问 InfluxDB 时不会存在跨域访问的限制，只要配置好数据源后，即可展示监控数据。

- 丰富的数据源接口，支持 InfluxDB、MySQL、ElasticSearch、PostgreSQL 等多数据源
- 丰富的 API 接口，方便自动化程序调用
- 监控 dashboard 导入导出，制作好模板后导入后修改参数即可实现实时监控
- 可方便的设置全局变量，动态变更整个 board 的显示参数
- 支持复杂的告警规则及邮件告警

# 常见问题

## grafana 视图曲线断线

### 现象

在并发量较大的情况下，所有 Grafana 视图都会发生不同程度的断线，对于采样统计的曲线会有剧烈下降的情况

### 分析

1. 发现 InfluxDB 所在主机的 CPU 满负荷运行，导致 Telegraf 发来的消息积压，严重时甚至宕机
2. 是由于 Grafana 某些视图的查询语句执行时，涉及的 Tag 设计有问题，导致统计算法性能极低
3. Tag 的值必须是枚举类型，而使用时用了与时间、自增 ID、UUID、多变参数相关的数值，导致 Tag 内容不断变化，作为索引性能极低

### 方案

- 重新设计 Tag，避免内容范围不断变化的字段作为 Tag
- 这种 Tag 不要作为 Grafana 视图的查询语句，只在特殊情况下酌情使用
