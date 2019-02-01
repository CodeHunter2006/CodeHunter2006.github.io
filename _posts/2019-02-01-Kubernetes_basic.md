---
layout: post
title:  "Kubernetes基本概念"
date:   2019-02-01 10:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/20190201_kubernetes_docker.jpg)
Dokcer为我们解决了环境搭建复杂、虚拟机重启速度慢等发布问题，并且通过DockerFile实现了对运行环境的版本控制、便于移植。正因为如此方便，所以我们的Docker往往会越来越多，组成分布式系统，当系统中的主机和容器数量达到一定程度时就会发生管理混乱。为了真正发挥Docker的效用，我们可以采用开源的Kubernetes容器管理系统。

Kubernetes简称K8S(中间正好8个字母)，是谷歌内部使用了十多年的一套分布式系统管理系统，后来开源出来。在希腊语中Kubernetes是"船长"的意思，如其名字，Kubernetes可以管理、调度分布式集群中的容器，让你在集群的海洋中也能找到方向。

# K8S的功能、特性
* 具有完备的集群管理能力
* 多扩多层次的安全防护和准入机制
* 多租户应用支撑能力
* 透明的服务注册和发现机制
* 內建智能负载均衡器
* 强大的故障发现和自我修复能力
* 服务滚动升级和在线扩容能力
* 可扩展的资源自动调度机制
* 多粒度的资源配额管理能力

# K8S系统结构图、构成要素说明
![K8S系统结构图](/assets/images/20190201_kubernetes_docker_3.png)

### Cluster(集群)
如果所示，集群就是一组节点(Node)，可以是物理主机或虚拟机，在其上安装了Kubernetes平台，可以对集群进行管理。

### Kubernetes Master
集群拥有一个Kubernetes Master（紫色方框）。Kubernetes Master提供集群的独特视角，并且拥有一系列组件，比如Kubernetes API Server。API Server提供可以用来和集群交互的REST端点。master节点包括用来创建和复制Pod的Replication Controller。

### Node(节点)
是物理或者虚拟机器，作为Kubernetes worker，通常称为Minion。

### POD
Pod安排在节点上，包含一组容器和卷。同一个Pod里的容器共享同一个网络命名空间，可以使用localhost互相通信。Pod是短暂的，不是持续性实体。
* 由于Pod是短暂的，可以用 __卷__ 来进行持久化
* 由于重启后IP地址可能改变，可以用 __Service__ 来进行查找

### Container(容器)
真正执行的程序就是跑在Container中的，通常就是Docker容器。

### Lable(标签)
一个Label是attach到Pod的一对键/值对，用来传递用户定义的属性。

可以对一个Pod贴多个Label，然后通过Selector选择特定的Pod，将Service或Replication Controller应用到上面。

### Replication Controller
确保任意时间都有指定数量的Pod"副本"在运行，如果监控有Pod发生问题，就立刻启动新的进行替换；如果Pod又恢复了运行，总数超出了设定，则进行关闭。

使用这种方法可以进行"滚动升级"(新版本逐步创建、旧版本逐步关闭)。

### Service
定义一系列Pod以及访问这些Pod的策略的一层抽象。Service通过Label找到Pod组。

首先每个Service要定义一个名称、绑定一些Pod，然后这个Service会提供DNS服务，只要查找特定名称就能找到可用的IP地址。并且Servie会对多台Pod提供透明的负载均衡，请求会被分发到代理(kube-proxy)完成。

有一个特别类型的Kubernetes Service，称为'LoadBalancer'，作为外部负载均衡器使用，在一定数量的Pod之间均衡流量。比如，对于负载均衡Web流量很有用。

# Rancher和K8S的关系
![K8S系统结构图](/assets/images/20190201_kubernetes_docker_4.jpg)
Rancher是一个开源的企业级容器管理平台。可以用可视化的方式对Kuberntes进行管理，并且提供了一些便利的工具，比自己逐步搭建Kubernetes要更容易。

# 学习资料：
![插图讲解K8S基本概念](/assets/images/20190201_kubernetes_docker_2.png)
推荐：插图讲解K8S基本概念<br/>
视频：[The Illustrated Children ‘s Guide to Kubernetes](https://v.qq.com/x/page/e0306wgo6t3.html)<br/>
翻译：[插画版Kubernetes指南（小孩子也能看懂的kubernetes教程）](https://www.cnblogs.com/kouryoushine/articles/8007648.html)

