---
layout: post
title: "Kubernetes常用Spec"
date: 2021-10-12 22:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/20190201_kubernetes_docker.jpg)
记录各种 Resource 类型的常用 Sepc(清单文件配置项)及常用 Describe 项。

## Service

提供一种抽象的"服务"概念，为一种服务提供一个唯一入口，其内部可以通过 LB 策略分配到实际的 Pod。

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

## Ingress

提供 Nginx 的功能：反向代理、7 层 LB。其本质就是通过 IngressController 监听配置规则生成 Nginx 的配置。
过多的向外暴露的 Service 会占用集群节点的端口，而对外端口资源是有限的，所以需要用 Ingress 转发。

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-http
  namespace: dev
spec:
  tls: # https 需要的密钥
    - hosts: # https 支持的 host 数组
        - nginx.test.com
      secretName: tls-secret # 指定密钥
  rules: # 规则数组
    - host: nginx.test.com # 转发的域名/IP
      http: # http 80 端口
        paths: # 转发 path 数组
          - path: /
            backend:
              serviceName: nginx-service # 内部 service 名
              servicePort: 80 # service 暴露的端口
```

- 创建密钥步骤：
  1. 生成证书(这里指自己测试用密钥，正式密钥需要正式第三方颁发的证书才可以)
     `openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -suj "/C=CN/ST=BJ/)-nginx/CN-test.com"`
  2. 创建密钥
     `kubectl create secret tls tls-secret --key tls.key --cert tls.crt`

```yml
# describe 项
Default backend: default-http-backend:80(<none>) # Nginx 对外提供的端口
Rules: # Host 映射规则
  Host           Path Backends
  ----           ---- --------
  nginx.test.com  /   nginx-service:80 (xx.xx.xx.xx:80,...)
```

## ReplicaSet(RS)

```yml
apiVersion: apps/v1 # 版本号
kind: ReplicaSet # 类型
metadata:
  name: rsname
  namespace: dev
  labels:
    controller: rs
spec:
  replicas: 3 # 副本数量，默认 1
  selector: # pod 选择器
    matchLabels: # Labels 匹配规则
      app: nginx-pod
    matchExpressions: # Expressions 匹配规则
      - { key: app, operator: In, values: [nginx-pod] }
  template: # 创建 pod 副本的模板
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - containerPort: 80
```

## Deployment(Deploy)

K8S 1.2 引入，提供一个部署流程，可以指定部署一系列资源进行部署。会启动 ReplicaSet 维护指定数量的 Pod。

- 支持 ReplicaSet 的所有功能
- 支持发布的停止、继续
- 支持版本的滚动更新和版本回退

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
  namespace: dev
spec:
  replicas: 3   # 目标是三个实例
  revisionHistoryLimit: 3 # 保留历史版本，默认 10
  paused: false # 暂停部署，默认 false
  progressDeadlineSeconds: 600 # 部署超时时间(s)，默认 600
  strategy: # 策略
    type: RollingUpdate # 滚动更新策略
    rollingUpdate:  # 滚动更新
      maxSurge: 30% # 最大额外可存在的副本数，可以为百分比或整数
      maxUnavailable: 30% # 最大不可用状态的 pod 数，可以为百分比或整数
  selector:
    matchLabels:
      app: nginx-pod
  template:   # 指定要部署的 pod 实例对应的模板
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

- revisionHistoryLimit
  保留历史版本以便进行版本回退，通过保留 RS 实现

- 调用`get`命令后各字段解释：
  - READY
    处于 READY 状态的 Pod 数量 / 总数量
  - UP-TO-DATA
    处于最新版本的 Pod 数量
  - AVAILABLE
    当前可用 Pod 数量

## DaemonSet

确保全部(或一些) Node 上运行一个 Pod 的副本。当有新 Node 加入 cluster 时，也会为他新增一个 Pod。
当有 Node 从集群移除时，Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

- 典型用法：
  - 在每个 Node 上运行日志收集或监控程序，如：filebeat、logstash、Prometheus Node Exporter
  - 在每个 Node 上定时拉取代码、镜像，已确保一定的缓存，避免冷启动速度慢

```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  labels:
    app: nginx
spec:
  template: # 同 Deployment，只是没有 replicas
```

## StatefulSet

适合处理"有状态"的 Pod，即 Pod 重启后名称、网络地址、存储不变。

- 和 Deployment 功能类似，但不依赖 ReplicaSet，而是直接控制 Pod
- 创建 Pod 时，自动为每个 Pod 实例创建唯一名称，序号`[0,n)`
- Pod 用 Headless 模式，直接通过内部域名访问，重启后域名不变
- PVC 的创建也是与 Pod 名称绑定，确保 Pod 重启后加载同一 PVC，需要设置 volumeClaimTemplates
- 用 ControllerRevision 进行多版本 template 管理，可以用于滚动升级，
- 升级时 StatefulSet 会按序号删除旧 Pod，自动创建同名新 Pod。默认每个 Pod 按顺序逐个升级。

```yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "nginx"
  replicas: 3 # default is 1
  podManagementPolicy: OrderedReady # 管理策略
  updateStragegy: RollingUpdate # 升级策略
  partition: 0 # default 0, old version pod num when rolling update
  revisionHistoryLimit: 10 # keep history to rollback, default 10
  template: # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: nginx
          image: k8s.gcr.io/nginx-slim:0.8
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # PVC templates
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnece"]
        storageClassName: "my-storage-class"
        resources:
          requests:
            storage: 1Gi
```

- Pod 管理策略
  - OrderedReady(默认值)
    在扩缩容时，要保证顺序。扩容时保证前面序号的 Pod 处于 Ready 状态后，再创建后面的 Pod；缩容时先删除序号最大的。
  - Parallel
    扩缩容 Pod 时无需等待。
- Pod 升级策略
  - RollingUpdate(默认)
  - OnDelete
    禁止主动升级，只有删除 Pod 后才自动用新版本创建
- partition
  升级新版本时保留旧版本 Pod 的数量，可用于**灰度升级**

- pod 名称：
  `nginx-web-0`
- pvc 名称：
  `www-storage-nginx-web-0`
- Pod 中用`controller-revision-hash`来标识当前版本

## Job

批量处理一次性任务。

- 当 Job 创建的 Pod 执行成功结束时，Job 将记录成功结束的 Pod 数量
- 当成功结束的 Pod 达到指定的数量时，Job 将完成执行

```yml
apiVersion: batch/v1 # 版本号
kind: Job
metadata:
  name:
  namespace:
  labels:
    controller: job
spec:
  completions: 3 # 指定需要成功运行 Pods 的次数，默认 1
  parallelism: 2 # 任意时刻并发运行的 Pod 的数量，默认 1(队列)
  activeDeadlineSeconds: 30 # 制定 Job 运行的时间限制(秒)，超时终止
  backoffLimit: 6 # 失败后重试次数，默认 6
  manualSelector: true # 是否可以使用 selector 选择 Pod，默认 false
  selector: # 指定控制器管理哪些 Pod
    matchLabels:
      app: counter-pod
    matchExpression: # Expressions 匹配
      - { key: app, operator: In, values: [counter-pod] }
  template: # 创建 Pod 副本的模板
    metadata:
      labels:
        app: counter-pod
    spec:
      restartPolicy: Never # 这里只能设置为 Never 或 OnFailure
      cantainers:
        - name: counter
          image: busybox:1.30
          command:
            [
              "bin/sh",
              "-c",
              "for i in 9 8 7 6 5 4 3 2 1; do echo $i;sleep 2;done",
            ]
```

## CronJob(CJ)

CronJob 可以控制 Job 间接管理 Pod 资源对象，可以指定时间点及重复运行的方式，相当于 Linux 中的 cron 任务。

```yml
apiVersion: batch/v1beta1 # 版本号
kind: CronJob
metadata:
  name:
  namespace:
  labels:
    controller: cronjob
spec:
  schedule: # cron 格式的运行时间点
  concurrencyPolicy: # 兵法之行策略，用于定义前一次作业运行尚未完成时的动作
  failedJobHistoryLimit: 1 # 为失败的任务保留历史记录数，默认为 1
  successfulJobHistoryLimit: 3 # 为成功执行的任务保留的历史记录数，默认为 3
  startingDeadlineSeconds: # 启动作业的超时时长(秒)
  jobTemplate: # 为 cronjob 定义 job 控制器模板
```

- concurrencyPolicy:
  - Allow
    允许 Jobs 并发运行(默认)
  - Forbid
    禁止并发运行，如果上一次运行尚未完成，则跳过下一次运行
  - Replace
    替换，取消当前正在运行的作业并用新作业替换它

# Storage

- Volume 是 Pod 中能够被多个容器访问的共享目录，被定义在 Pod 上，被多个容器挂载到具体的文件目录下。
- Volume 类型：
  - 简单存储：EmptyDir、HostPath、NFS
  - 高级存储：PV、PVC
  - 配置存储：ConfigMap、Secret

## EmptyDir

EmptyDir 是在 Pod 被分配到 Node 时创建的，初始值为空，K8S 会自动分配一个目录，当 Pod 销毁时，EmptyDir 中的数据也会被删除。

- 可作为应用临时存储空间
- 多个 Container 共享数据

EmptyDir 是直接定义在 Pod spec 中的：

```yml
apiVersion: v1
kind: Pod
metadata:
  name: volume-emptydir
  namespace: dev
spec:
  containers:
    - name: nginx
      image: ngxin:1.17.1
      ports:
        - containerPort: 80
      volumeMounts: # 将 pod 生命的 logs-volume 挂载到 /var/log/nginx
        - name: logs-volume
          mountPath: /var/log/nginx
    - name: busybox
      image: busybox:1.30
      command: ["/bin/sh", "-c", "tail -f /logs/access.log"] # 动态读取指定内容
      volumeMounts: # 将 logs-volume 挂载到 /logs
        - name: logs-volume
          mountPath: /logs
  volumes: # 声明 volume
    - name: logs-volume
      emptyDir: {} # 指定类型，无需具体参数
```

## HostPath

将 Node 主机中一个实际目录挂载到 Pod 中，Pod 销毁时数据不销毁。

```yml
# 同 EmptyDir 示例
volumes:
  - name: logs-volume
    hostPath:
      path: /root/logs
      type: DirectoryOrCreate # 目录存在就使用，不存在就先创建后再使用
```

- type:
  - DirectoryOrCreate
    目录存在就使用，不存在就先创建后再使用
  - Directory
    目录必须存在，不自动创建
  - FileOrCreate
    文件，同 DirectoryOrCreate
  - File
    文件必须存在
  - Socket
    unix 套接字必须存在
  - CharDevice
    字符设备必须存在
  - BlockDevice
    块设备必须存在

## NFS(Net File System)

NFS 是一个网络文件存储系统，可以搭建一台 NFS，然后 Pod 中存储直接连接到 NFS 系统上，这样可以避免 Pod 迁移到其他 Node 上存储丢失的情况。

```yml
# 同 EmptyDir 示例
volumes:
  - name: logs-volume
    nfs:
      server: xxx.xxx.xx.xx # nfs服务器地址
      path: /root/data/nfs # 共享文件路径
```

## PersistentVolume(PV)

PV 是集群级别的资源，为底层存储提供了一层抽象，可以绑定到容器上。
PV 可由对存储比较了解的 K8S 管理员设置，PVC 可由开发人员直接使用，这样降低了各自的复杂度。

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  nfs: # 存储类型，与底层存储对应
    path: /root/data/pv1
    server: xxx.xxx.xxx.xxx
  capacity: # 存储能力，目前只支持存储空间的设置
    storage: 1Gi
  accessMode: # 访问模式
    - ReadWriteMany
  storageClassName: xxx # 存储类别
  persistentVolumeReclaimPolicy: Recycle # 回收策略
```

- 存储类型
  底层存储类型不同，对应的 spec 子选项会有不同
- 存储能力
  目前只有存储容量设置，未来可能有 IOPS 等指标
- 访问模式
  - ReadWrightOnce(RWO): 读写，但是只能被单个节点挂载
  - ReadOnlyMany(ROX): 只读，可以被多个节点挂载
  - ReadWriteMany(RWX): 读写，可以被多个节点挂载
- 回收策略
  当 PV 不再被使用后的处理方式
  - Retain(保留)，保留数据，需要管理员手动清理
  - Recycle(回收)，清除 PV 中的数据，相当于`rm -rf /thevolume/*`
  - Delete(删除)，常见于云服务商的存储服务
- 存储类别

  - 具有特定类别的 PV 只能与请求了该类别的 PVC 进行绑定
  - 未设定类别的 PV 只能与不请求任何类别的 PVC 进行绑定

- 状态
  一个 PV 的生命周期可能处于四种不同阶段：
  - Available(可用)：可用状态，未被绑定
  - Bound(已绑定)：表示 PV 已被 PVC 绑定
  - Released(已释放)： 表示 PVC 被删除，但是资源还未被集群重新声明
  - Failed(失败)：表示该 PV 的自动回收失败

```yml
# describe
status: Bound # 绑定状态
claim: namespace1/pvc1 # 绑定到某命名空间下的某个 pvc
reason: #
```

- status
  - Bound 已绑定
  - Available 待绑定

## PersistentVolumeClaim(PVC)

PVC 用于向 cluster 申请存储。
由于 PVC 的 spec 和 PV 很相近，所以不再解释。

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
  namespace: dev
spec:
  accessModes:
  selector: # 采用 label 对 PV 选择
  storageClassName:
  resources: # 请求空间
    requests:
      storage: 1Gi
```

```yml
# describe

status: Pending # 绑定状态
```

- status
  - Pending 未找到合适的 PV，挂起
  - Bound 已绑定到 PV

## ConfigMap

用来存储配置信息，key 对应文件、value 对应文件内容。
configMap 支持动态更新(需要一定时间)。

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap
  namespace: dev
data: # 对应的数据
  info1: | # 文件名，下面是文件内容
    field1: 0
    field2: hello
  info2: |
    xxxyyyzzz
---
kind: Pod
spec:
  containers:
    - name: nginx
      volumeMounts: # configmap 挂载目录
        - name: config
          mountPath: /configmap/config
  volumes:
    - name: config
      configMap:
        - name: configmap
```

## Secret

和 configMap 功能类似，可以对数据进行 base64 编码保存、自动解码，**并不是真正的加密**。

- 使用流程：
  1. 创建 secret 资源，将 value 内容以 base64 编码
  2. 创建 Pod，绑定 secret 为 volume
  3. 在 Pod 中，直接读取 secret 对应的 key(读取文件)，可以自动返回解码后的原始数据内容

```yml
apiVersion: v1
metadata:
  name: secret
  namespace: dev
type: Opaque
data:
  username: YWRtaW4= # 原文 "admin" 经 base64 编码后的结果
  password: MTIzNDU2
---
kind: Pod
spec:
  containers:
    - name: nginx
      volumeMounts: # secret 挂载目录
        - name: config
          mountPath: /secret/config
  volumes:
    - name: config
      secret:
        - secretName: secret
```

```yml
# describe
Data
====
password: 6 bytes # 将真实内容隐藏
username: 5 bytes
```

# Pod

```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
spec:
  containers: # container 数组
    - name: nginx1
      image: nginx:1.0
      cmmand: ["/bin/sh", "-c", ";"]
      volumeMounts:
        - name: volume1
          mountPath: /root/
  volumes:
    - name: volume1
      persistentVolumeClaim:
        claimName: pvc1
        readOnly: false
```