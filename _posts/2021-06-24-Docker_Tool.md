---
layout: post
title: "Docker 一些辅助工具"
date: 2021-06-24 23:00:00 +0800
tags: Docker
---

# runlike

runlike 是一个基于 python 的工具，可以方便的查看 container 启动参数。

- 安装
  `pip install runlike`

- 查看 container 启动参数
  `runlike -p <容器名>|<容器ID>`

# docker-compose

一个用来定义和运行复杂应用的 docker 工具。使用 docker-compose 后无需再用 shell 来启动 docker 容器，并且可以同时启动多个 docker 容器进行配合。

- 在 docker-compose 中，以 service 来定义 docker 的配置，service 可以启动、停止、重启。

- 安装
  `sudo pip install docker-compose`

- docker-compose 文件结构：

```yml
# docker-compose.yml
version: "3"
services:
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```
