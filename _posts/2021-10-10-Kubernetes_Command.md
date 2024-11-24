---
layout: post
title: "Kubernetes常用命令"
date: 2021-10-10 22:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/20190201_kubernetes_docker.jpg)
记录 K8S 常用命令，练习命令可以用[minikube](https://minikube.sigs.k8s.io/docs/start/),
命令参考[kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

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

- `--dry-run=client -oyaml` 命令从并不从客户端真正发出，用来查看发出的命令
- `--dry-run=server -oyaml` 命令在服务端不进行持久化，用来检查命令是否有逻辑错误

# 基本配置

- 安装 kubectl 工具：`brew install kubernetes-cli helm`，这个命令可以将 helm 工具顺便安装好。

- 连接 API-Server 需要配置好"cube config"文件，在 mac 中在`~/.cube/config`。

  - 默认将使用这个配置文件，如果需要利用其他配置文件，可以用设置参数`kubectl --kubeconfig xxx get namespace`

- 可以用`lens`代替`dashboard`提供 UI，`brew install --cask lens`

# 常用命令

## api-versions

查看 K8S 集群内所有资源对应的版本号

`kubectl api-versions`

## api-resources

查看 K8S 集群内所有可以支持的资源类型

`kubectl api-resources`

## get

查询某种资源列表，也可以同时查询多种，用','隔开。

- `-o yaml` 指定显示的格式

  - `wide` 显示更多列信息
  - `yaml` 以 yaml 显示更详细信息
  - `json` 以 json 显示

- `-w` 显示了当前内容后继续`watch`变化。如果在命令中指定了 name，则新增资源不会被 watch 到。
- `--show-labels` 显示标签
- `-A` 显示所有 namespace 下的资源，在不知道具体 namespace 情况下可以使用，列表中有 namespace 列

`kubectl get ns`
获取 namespace 列表

`kubectl -n xxx get pod`
获取 xxx namespace 下的 pod 列表

`kubectl get pod <pod_name> -n xxx -o yaml`
以 yaml 格式展示结果，内容非常详细。可以指定具体实例名。

## create

`kubectl create xxx_name --image=xxx`
创建一个 pod，运行一个 container。这个操作本质上是创建了一个 deployment，`xxx_name`就是 deployment 的名称。

- `-- record`
  记录创建过程的更新信息

## apply

创建或更新资源配置文件，在配置文件中指定资源类型和参数，文件中可以分多段，也可以直接执行文件夹。

`kubectl apply -f nginx-pod.yaml`
声明式配置资源

## describe

`kubectl describe pod pod_name -n xxx`
获取某种指定负载对象的详细信息(spec、运行时)，包括资源的创建时间、事件、标签、注解等更多的细节信息。

- 一个 workload(deployment、pod、service...) 启动后会自然产生一系列运行时信息，可以用 describe 查看
  - 如 pod 的`state`、`event`
  - 如 deployment 的 `Replicas: 2 current / 2 desired`

## edit

编辑对应 Resource 的 Spec 文件

`kubectl edit deploy deploy-name -n xx`
用 vi 打开界面编辑 Deployment，保存后执行。

## exec

`kubectl -n xxx exec -it testpod -- bash`
登录到指定的 pod 执行命令

`kubectl -n xxx exec -it testpod -c testcontainer`
登录到指定的 container 执行命令

## explain

分层说明某个资源的 sepc 项

`kubectl explain pod`
显示一级属性说明

`kubectl explain pod.metadata`
解释`metadata`属性及其二级属性

## expose

自动创建一个 serveice(svc)，向外暴露端口。

`kubectl expose deployment hello-minikube --type=NodePort --port=8080`
向外暴露 Application 到 8080 端口

## label

标签相关操作

`kubectl label pods foo unhealthy=true`
为某个 pod 第一次设置一个 label

- `--overwrite` 覆盖写入
- `--all` 不指定 resourceName 设置到所有资源
- `--resource-version=1` 指定 resource-version 下才起作用，避免冲突

`kubectl label pods foo bar-`
删除 label

## log

`kubectl logs pod testpod`

查看标准输出

## rollout

对 Controller 下 Pod 进行批量滚动操作，可作用于 deployment/replicaset/deamonset

`kubectl rollout undo deploy deploy-name --to-revision=1 -n dev`
将版本回退到"revision 1"

- `status`
  显示当前升级状态
- `history`
  显示升级历史记录
  - `--revision=xxx`
    查看历史版本的具体 yaml 内容
- `pause`
  暂停版本升级过程
- `resume`
  继续已经暂停的版本升级过程
- `restart`
  滚动重启
- `undo`
  回滚到上一级版本(可以使用`--to-revision` 回滚到指定版本，`status`可查看 revision 号)

## run

自动创建一个 Deployment 运行指定 镜像。

`kubectl run deploy-name --image=nginx:1.17.1 --requests=cpu=100m -n dev`

## cordon/uncordon

`kubectl cordon $NODENAME`
将 Node 从调度中独立出来，以避免这个 Node 中的 Pod 运行状态发生变更，可用于调错时保存现场。

## port-forward

将 K8S 集群内部端口暴露在 localhost

`kubectl port-forward -n namespace svc/mysql 3307:3306`
打通 localhost:3307 和目标的 3306 端口

`kubectl port-forward -n namespace pod/xxx 123 456 789`
同时转发三个端口，localhost 和 target 端口号相同

- `svc/xxx` 转发 service
- `pod/xxx` 转发 podName 对应的 pod
- `deploy/xxx` 转发 deployment
- `rs/xxx` 转发 rs

## scale

扩/缩容

`kubectl scale deploy deploy-name --replicas=5 -n dev`
变更 Deployement 的 Pod 数量

## set

设置 spec 的值

`kubectl set image statefulset nginx-web nginx=nginx:mainline`
向名为"nginx-web"设置新版本的 Pod 镜像，进行升级。

## taint

设置污点

`kubectl taint nodes node1 key=value:effect`
设置污点

`kubectl taint nodes node1 key:effect-`
去除污点

`kubectl taint nodes node1 key-`
去除所有污点

## top

查看资源使用情况

`kubectl top pod -n dev`
查看 Pod 资源的 CPU、Memory 使用情况

- cpu 的单位 1CPU = 100m

## cp

在当前主机和容器间拷贝文件

- `kubectl cp -n namespace [-c containerName] podName:relativePath targeFilePath`
  拷贝容器中的文件到本机

  - 如果只有一个 container，则无需填`-c containerName`
  - podName 后的地址是相对地址，不能以`/`开头
  - 相对地址是基于`WORKDIR`开始的

- 从本机拷贝到容器中只要调换参数即可

## 其他

`kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4`
部署一个 Application

`kubectl rolling-update redis -f redis-rc.update.yaml`
滚动升级

`kubectl get endpoints`
显示 svc 和对应接入点(ip+port)的对应关系

# 命令场景

## pod 启动后未 ready，需要到 container 查看

1. `kubectl describe pod xxx`
   查看 pod 在哪些节点有实例
2. 来到对应 node `docker ps|grep xxx`
   找到对应的 container
3. `docker exec -it xxx bash` 进入查看
