---
layout: post
title: "Kubernetes Context"
date: 2021-12-13 22:00:00 +0800
tags: K8S
---

在 K8S 操作时，经常需要在多个 cluster/namespace/user 之间切换，为了降低命令的复杂度，K8S 命令行工具提供了 context 功能。
选中 context 后，执行`kubectl xxx`命令时可以自动填充一些参数(cluster/namespace/user)。

# context 文件结构

通常 context 和 k8s cluster 信息都存放在`~/.kube/config`文件中

```yml
apiVersion: v1
current-context: context1 # 当前选中的 context。如果没有则无法 K8S 操作
contexts: # 上下文信息，name 和 cluster 是关键属性
  - context:
      cluster: cluster1
      user: cluster1-developer
    name: context1 # context 名称
  - context:
      cluster: cluster2
      user: cluster2-developer
      namespace: name2
    name: context2
clusters: # 集群信息
  - cluster:
      certificate-authority-data: xxx
      server: xxx
    name: cluster1
  - cluster:
      certificate-authority-data: xxx
      server: xxx
    name: cluster2
kind: Config # 本文件类型
preferences: {}
users: # user 信息
  - name: xxx
    user:
      exec:
```

# 常用命令

- `kubectl config set-context context1 --namespace=development --cluster=cluster1 --user=user1`
  创建 context 可选参数为`namespace` `cluster` `user`

- `kubectl config get-contexts`
  显示 context 列表

- `kubectl config current-context`
  显示当前选中的 context

- `kubectl config use-context context1`
  选择 context
