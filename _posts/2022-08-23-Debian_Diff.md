---
layout: post
title: "Debian 差异"
date: 2022-08-23 22:00:00 +0800
tags: Linux
---

![Debian](/assets/images/2022-08-23-Debian_Diff_1.jpeg)
记录 Debian 系统差异点。

# CMD

- `ip address` 查看本机 IP 地址

- `ip route` 查看路由表

- `ss` 查看端口占用

- `/etc/init.d/networking restart` 重启网络
- `systemctl restart network` 重启网络

- `systemctl status networking.service` 查看网络启动失败原因

- `ip route add 192.168.0.0/24 via 192.168.0.1 dev ethn metric 1000` 对网卡添加路由
  - `192.168.0.0/24`以表示掩码占 24 字节，`192.168.0.0`~`192.168.0.255`的网段
  - `192.168.0.1`表示这个网段的要走哪个网关
  - `ethn` 是网卡名
  - `1000` 表示到达目的地的跳跃点数，越大优先级越低
- `ip route del 192.168.0.0/24 via 192.168.0.1 dev ethn` 删除上面创建的这条路由
