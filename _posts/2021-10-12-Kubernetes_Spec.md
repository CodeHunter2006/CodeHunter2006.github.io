---
layout: post
title: "Kubernetes常用Spec"
date: 2021-10-12 22:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/20190201_kubernetes_docker.jpg)
记录各种 Resource 类型的常用 Sepc(清单文件配置项)及常用 Describe 项。

## Service

```yml
kind: Service # 资源类型
apiVersion: v1 # 资源版本
metadata: # 元数据
  name: service # 资源名称
  namespace: dev # 命名空间
spec: # 描述
  selector: # 标签选择器，用于确定当前 service 代理哪些 pod
    app: nginx
  type: # service 类型，指定 service 的访问方式
  clusterIP: # 虚拟服务的 ip 地址
  sessionAffinity: # session 亲和性，支持 ClientIP、None 两项
  ports: # 端口信息
    protocol: TCP
    port: 3017 # service 自己对外提供的端口
    targetPort: 5003 # pod 对外提供的端口
    nodePort: 31122 # 主机端口
  externalName: www.baidu.com # 外部服务地址或域名
```

```yml
# describe 项

Endpoints: 10.xxx.xx.xx,... # Service 对应的覆盖到的 Pods IP
Session_Affinity: None # 亲和属性
```

### 不同 type 对应的配置和功能

- type 可选项:
  - ClusterIP(默认值)
    它是 K8S 系统自动分配的虚拟 IP，只能在集群内部访问。
    - 可选设置 spec `clusterIP`，如果不设置则自动分配
  - NodePort
    将 Service 通过指定的 Node 上的端口暴露给外部，通过此方法，可以在集群外部访问服务
    - 可设置`nodePort`，范围是`[30000, 32767]`。如果不设置，则自动分配一个该范围内的端口
  - LoadBalancer
    使用外接负载均衡器完成到服务的负载分发，此模式需要外部云环境支持。一般在 K8S 内，通过 Ingress 实现负载均衡和反向代理。
  - ExternalName
    把集群外部的服务引用集群内部，在内部直接使用。
    - 需设置`externalName`，可以是 IP 或域名。
  - None
    Headless 模式，不指定 IP，直接根据 svc 名，用内部域名的形式访问。
    `servicename.svc.cluster.local`，其中`svc.cluster.local`是可以在集群配置的

## Deployment

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
  namespace: dev
spec:
  replicas: 3   # 目标是三个实例
  selector:
    matchLabels:
      app: nginx-pod
  template:   # 指定要部署的实例对应的模板
    metadata: # 要部署的实例的元数据
      labels: # 制定多个 label
        app: nginx-pod
    spec:
      containers:   # 要部署的容器
        - name: nginx
          image: nginx: 1.17.1  # 镜像版本
          ports:
          - containerPort: 80   # 对容器外部开放的端口号
```
