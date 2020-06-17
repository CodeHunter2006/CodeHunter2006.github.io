---
layout: post
title: "Network Protocols"
date: 2020-06-05 22:00:00 +0800
tags: Server
---

![Fitness](/assets/images/2020-06-05-Network_Protocol_1.jpeg)

# 网络传输可能发生的问题：

- 数据丢包
- 数据重复
- 数据完整性校验
- 数字信号转模拟信号
- 不同介质间的转换
- 信号衰减

# 通过网络分层解决网络可靠性问题

将网络分层，上下两层之间保持接口不变，这样可以修改、替换其中一层，不会影响其他层。
常用的网络分层：

- OSI(Open System Interconnection Reference Model)开放系统互联参考模型
- TCP/IP 协议族，OSI 的表示、会话两层被合并入应用层

# 一个 HTTP 请求

1. 在浏览器输入网址按下回车
2. 通过域名查找 DNS 得到 IP 地址

# 常见问题

- 为什么 TCP 保证了传输数据的正确性，还需要在应用层加 CRC 或 MD5 校验？
  因为处于不同的层。TCP 在传输层保证正确性，但是应用层收到数据后可能未正常保存就崩溃了，这种情况下在 TCP 层看到是正常传输完毕的，而最终文件却有问题，需要校验。
