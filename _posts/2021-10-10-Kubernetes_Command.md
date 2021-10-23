---
layout: post
title: "Kubernetes常用命令"
date: 2021-10-10 22:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/20190201_kubernetes_docker.jpg)
记录 K8S 常用命令，练习命令可以用[minikube](https://minikube.sigs.k8s.io/docs/start/)

# 相关概念

- 在 K8S 中**一切皆资源**，我们的命令本质上是对资源的操作或配置。

- 三种资源管理方式：

  - 命令式对象管理：
    `kubectl run nginx-pod --image=nginx:1.15.1 --port=80`
  - 命令式对象配置：
    `kubectl create -f nginx-pod.yaml`
    - 会覆盖对象原有参数，所以在配置文件中要包含全部配置参数
  - 声明式对象配置：
    `kubectl apply -f nginx-pod.yaml`
    - 如果没有资源则创建、如果已有则更新配置
    - 重复`apply`时，由于重复设置，所以不会起作用

| 类型           | 操作对象 | 适用环境 | 优点           | 缺点                             |
| -------------- | -------- | -------- | -------------- | -------------------------------- |
| 命令式对象管理 | 对象     | 测试     | 简单           | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 | 文件     | 开发     | 可以审计、跟踪 | 项目大时，配置文件较多，操作麻烦 |
| 声明式对象配置 | 目录     | 开发     | 支持目录操作   | 意外情况下难以调试               |

- kubectl 命令语法：
  `kubectl [command] [type] [name] [flags]`
  一般前两个参数是必须的，后面是可选的。

  - command
    指定要对资源执行的操作，如 create、get、delete
  - type
    指定资源类型，如 deployment、pod、service
  - name
    指定资源的名称，如果不指定名称，则针对整个类型执行，如`get`获取该类型的列表
  - flag
    指定额外的可选参数，如 `-l`

- 可以用`kubectl --help`查询可用 command，列表自带分类。

- 注意默认情况下命令都是针对`default` namespace 执行的，一般都要设置`--namespace`参数，缩写为`-n`

- 可以用`kubectl api-resources`查看 K8S 中的资源，可以看到"SHORTNAMES"表示其缩写，在命令中可以利用。

- 注意`kubectl`命令对资源名称结尾的`s`并不强制要求，可以带也可不带

# 基本配置

安装 kubectl 工具：`brew install kubernetes-cli helm`，这个命令可以将 helm 工具顺便安装好。

连接 API-Server 需要配置好"cube config"文件，在 mac 中在`~/.cube/config`。

# 常用命令

## get

查询某种资源列表

- `-o yaml` 指定显示的格式
  - `wide` 显示更多列信息
  - `yaml` 以 yaml 显示更详细信息
  - `json` 以 json 显示

`kubectl get ns`
获取 namespace 列表

`kubectl -n xxx get pod`
获取 xxx namespace 下的 pod 列表

`kubectl get pod -n xxx -o yaml`
以 yaml 格式展示结果，内容非常详细

## create

`kubectl create xxx_name --image=xxx`
创建一个 pod，运行一个 container。这个操作本质上是创建了一个 deployment，`xxx_name`就是 deployment 的名称。

## apply

创建或更新资源配置文件，在配置文件中指定资源类型和参数，文件中可以分多段，也可以直接执行文件夹。

`kubectl apply -f nginx-pod.yaml`
声明式配置资源

## describe

`kubectl describe pod xxx -n xxx`
获取指定 pod 的详细信息。其中的`state`、`event`是比较重要的调试信息。

## log

`kubectl logs pod testpod`

查看标准输出

## exec

`kubectl -n xxx exec -it testpod bash`
登录到指定的 pod 执行命令

`kubectl -n xxx exec -it testpod -c testcontainer`
登录到指定的 container 执行命令

## cordon/uncordon

`kubectl cordon $NODENAME`
将 Node 从调度中独立出来，以避免这个 Node 中的 Pod 运行状态发生变更，可用于调错时保存现场。

## 其他

`kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4`
部署一个 Application

`kubectl expose deployment hello-minikube --type=NodePort --port=8080`
向外暴露 Application 到 8080 端口

`kubectl scale rc redis --replicas=3`
执行扩容缩容

`kubectl rolling-update redis -f redis-rc.update.yaml`
滚动升级
