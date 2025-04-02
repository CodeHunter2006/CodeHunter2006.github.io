---
layout: post
title: "Kubernetes常用Spec"
date: 2021-10-12 22:00:00 +0800
tags: Docker K8S HighConcurrency
---

![kubernetes_docker](/assets/images/20190201_kubernetes_docker.jpg)
记录各种 Resource 类型的常用 Sepc(清单文件配置项)及常用 Describe 项。

# Pod

- Pod 中可能有多个容器，至少有一个根容器，根容器的作用：
  - 以它为依据，评估整个 Pod 的健康状态
  - 可以再跟容器上设置 IP 地址，其他容器都以此 IP 实现 Pod 内部的网络通信(容器分配不同 Port)
- Pod 内部的通讯采用虚拟二层网络技术实现，例如 Flannel

```yml
# kubectl explain pod
apiVersion: <string> # K8S 资源版本号，必须可以用 `kubectl api-versions` 查询到
kind: <string> # 类型，由 K8S 内部定义，必须可以用 `kubectl api-resources` 查询到
metadata: <Object> # 元数据，主要是资源标识和说明，常用的有 name、namespace、labels 等
spec: <Object> # 描述，是配置中最重要的部分，对各种资源配置详细描述
status: <Object> # 状态信息，内容无需定义，由 kubernetes 自动生成
```

```yml
# kubectl explain pod.spec
containers: <[]Object> 容器列表，用于定义容器的详细信息
initContainers: <[]Object> 初始化容器列表
nodeName: <string> 根据nodeName值将pod调度到指定node节点
nodeSelector: <map> 同上，用于调度到满足条件的 node 节点
hostNetwork: <boolean> 是否使用主机网络模式，默认为 false，如果设置为 true，表示使用宿主机网络
volumes: <[]Object> 存储卷，用于定义 pod 上挂载的存储
restartPolicy: <string> 重启策略，表示 Pod 在遇到故障时候的处理策略
```

```yml
# 示例
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
  labels:
    type: cnchTSO
    user: username
spec:
  containers: # container 数组
    - name: nginx1
      image: nginx:1.17.0
      imagePullPolicy: IfNotPresent
      command: ["/bin/sh", "-c", ";"]
      env: # 环境变量由多个 key-value 组成
        - name: "username" # 环境变量名
          value: "admin" # 环境变量值
        - name: "password"
          value: "123456"
      volumeMounts:
        - name: volume1
          mountPath: /root/
  volumes:
    - name: volume1
      persistentVolumeClaim:
        claimName: pvc1
        readOnly: false
```

- get 命令返回结果说明

  - READY
    pod 中 container 的就绪状态`就绪数/总数`
  - STATUS
    pod 状态
  - RESTARTS
    pod 重启次数
  - AGE
    pod 创建后的存在时间。重启不影响 AGE

- 可以通过 `describe` 命令结果中的**Events**查看运行情况的事件

```yml
# describe
IP: xxx.xxx.xx.xx # K8S 分配的内网 IP
Events: # 记录 pod 运行中的一系列关键事件
  Type(事件类型) Reason(触发动作) Age(发生时点) From(事件来源) Message(事件内容)
```

## container 设置

```yml
# kubectl explain pod.spec.containers
name: <string> # 容器名称
image: <string> # 容器需要的镜像地址
imagePullPolicy: <string> # 镜像拉取策略
command: <[]string> # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
args: <[]string> # 容器的启动命令需要的参数列表
env: <[]Object> # 容器环境变量的配置
ports: <[]Object> # 容器需要暴露的端口号列表
resource: <Object> # 资源限制的资源请求的设置
livenessProbe: <Object> # 存活性探针
readinessProbe: <Objet> # 就绪性探针
```

- imagePullPolicy 镜像拉取策略

  - Always
    总是从远程仓库拉取镜像
  - IfNotPresent
    本地有则优先用本地的，本地没有则用远程的
  - Never
    只使用本地的，从不去远程仓库拉取，本地没有就报错
  - 如果 image 填写了具体的版本号，则默认值为 IfNotPresent
  - 如果 image 填写的 latest 或未填写(等价于 latest)，则默认值为 Always

- 问题：command 本身是字符数组，可以满足命令+参数的功能，为什么还需要 args？
  这两项在 spec 中其实是覆盖实现 Dockerfile 中 ENTRYPOINT 的功能。

  - 如果 command 和 args 都没写，那么用 Dockerfile 的配置
  - 如果只写了 command，那么 Dockerfile 默认的配置会被忽略，执行 command
  - 如果只写了 args，那么 Dockerfile 中配置的 ENTRYPOINT 命令会被执行，使用 args 作为参数
  - 如果 command 和 args 都写了，那么 Dockerfile 的配置被忽略，执行 command 并追加 args 参数

- 技巧：如何实现 pod 启动后进入等待避免`crash`?
  - 启动命令设置为`command: ["/bin/sh", "-c", "tail -f /dev/null"]`，这样就可以一直等待
  - 为了避免重新创建 pod，要把`readinessProbe`设置为合理参数

```yml
# kubectl explain pod.spec.containers.ports
name: <string> # 端口名称，如果指定，必须保证 name 在 pod 中是惟一的
containerPort: <integer> # 容器要监听的端口 (0, 65536]
hostPort: <integer> # 容器要在主机上公开的端口，如果设置，主机上只能运行容器的一个副本(一般不设置)
hostIP: <string> # 要将外部端口绑到的主机 IP (一般不设置)
protocol: <string> # 端口协议。必须是 UDP/TCP/SCTP。默认为TCP。
```

- 一般只设置`containerPort`即可
- 访问容器中的程序时，可以用`podIP:containerPort`

```yml
# kubectl explain pod.spec.containers.resources
resources: # 资源配额
  request: # 要求的资源下限，不满足则无法运行
    cpu: "1" # CPU core 数，可以为整数或小数
    memory: "10Mi" # 内存容量
  limits: # 资源上限
    cpu: "2"
    memory: "1Gi"
```

- memory 可选项：Gi、Mi、G、M 等

## 生命周期

- pod 创建
- 运行初始化容器(init container)
  - 初始化容器提供主容器镜像中不具备的工具程序或自定义代码，可为主容器创造依赖条件
  - 初始化容器必须按照定义的顺序执行，直至结束，如果中间某个失败，K8S 会一直重启直到完成
- 运行主容器(main container)
  - 容器启动后钩子(post start)、容器终止前钩子(pre stop)。钩子的三种执行方式(配置 container.lifecycle)：
    - `exec` 执行指定的命令，以命令返回 exitCode 为准，0 为成功。如`cat /test/testfile.txt`
    - `tcpSocket` 访问指定的 tcp socket，如果可以建立连接则算成功。
    - `httpGet` 访问指定的 http 网址，如果返回状态码在`[200,399]`则算成功。
  - 容器活性监测(liveness probe)、就绪性探测(readiness probe)。
    探针的探测方式和上面钩子函数的方式相同，有三种方法。
    如果经过探测，pod 状态不符合预期，那么 K8S 会"摘除"该实例，不再承担业务流量。两种探针：
    - liveness probes: 存活性探针，用于检测应用实例当前是否处于正常运行状态，如果不是，则 K8S **重启容器**
      - 可以访问特定 url 判断是否存活
    - readiness probes: 就绪性探针，用于检测应用实例当前是否可以接收请求，如果不能，K8S 不会**转发流量**
      - 可以访问本机的特定 url 判断服务是否就绪
- pod 终止

```yml
# kubectl explain pod.spec.containers.liveneesProbe
exec <Object>     # 执行指定的命令
tcpSocket <Object>  # 连接特定的 TCP
httpGet <Object>  # 访问指定的 http/https url
initialDelaySecond <integer>  # 容器启动后等待多少秒执行第一次探测
timeoutSeconds <integer>  # 探测超时秒数。默认 1 秒，最小 1 秒
periodSeconds <integer>   # 执行探测的间隔秒数。默认 10 秒，最小 1 秒
failureThreshold <integer>    # 连续探测失败多少次才被认定为失败。默认 3， 最小 1
successThreshold <integer>    # 连续探测成功多少次才被认定为成功。默认 1
```

- 在整个生命周期中， pod 会出现 5 种状态(相位)：

  - Pending(挂起)：apiserver 已创建了 pod 资源对象，但它尚未被调度完成或仍处于下载镜像过程中
  - Running(运行中)：pod 已经被调度至某节点，并且所有容器都已经被 kubelete 创建完成
  - Succeeded(成功)：pod 中的所有容器都已经成功终止并且不会被重启
  - Failed(失败)：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非 0 的退出状态
  - Unknown(未知)：apiserver 无法正常获取到 pod 对象的状态信息，通常由网络通信失败导致

- pod 创建过程：

  1. 用户通过 kubectl 或其他 api 客户端提交需要创建的 pod 信息给 apiServer
  2. apiServer 开始生成 pod 对象的信息，并将信息存入 etcd，然后返回确认信息至客户端
  3. apiServer 开始反映 etd 中的 pod 对象变化，其它组件用 watch 机制来跟踪检查 apiServer 上的变动
  4. scheduler 发现有新的 pod 对象要创建，开始为 pod 分配主机并将结果信息更新至 apiServer
  5. node 节点上的 kubelet 发现有 pod 调度过来，尝试调用 docker 启动容器，并将结果回送至 apiServer
  6. apiServer 将接收到的 pod 状态信息存入 etcd 中

- pod 终止过程：

  1. 用户向 apiServer 发送删除 pod 对象的命令
  2. apiServer 中的 pod 对象信息会随着时间的推移而更新，在宽限期内(默认 30s)，pod 被视为 dead
  3. 将 pod 标记为 terminating 状态
  4. kubelet 在监控到 pod 对象转为 terminating 状态的同时启动 pod 关闭过程
  5. 端点控制器在监控到 pod 对象的关闭行为时将其从所有匹配到此端点的 service 资源的端点列表中移除
  6. 如果当前 pod 对象定义了 preStop 钩子处理器，则在其标记为 terminating 后即会以同步的方式启动执行
  7. pod 对象中的容器进程收到**停止**信号
  8. 宽限期结束后，若 pod 中还存在仍在运行的进程，那么 pod 对象会收到**立即终止**的信号
  9. kubelet 请求 apiServer 将此 pod 资源的宽限期设置为 0 从而完成删除操作，此时 pod 对于用户已不可见

- 重启策略(pod.spec.restartPolicy)
  当存活性探针检测到失败时，就需要重启 container，这时候要依照重启策略执行。
  - 重启策略选项：
    - Always(默认值)
      容器失效时，自动重启该容器
    - OnFailure
      容器终止运行且退出码为"非 0"时重启
    - Never
      不重启
  - 重启策略适用于 pod 对象中的所有容器，首次需要重启的容器，将立即进行重启。为了防止不断重启造成性能问题，
    如果重启失败将按照退让策略尝试重启，退让延迟时长每次分别为 10s、20s、40s、80s、160s、300s，
    达到 300s 后将不再增大延迟，每隔 300s 重试

## 调度

默认情况下，Pod 在哪个 Node 节点运行是由 Scheduler 组件采用相应的算法计算出来的，这个过程不收人工控制。
有些情况下我们想人工控制 Pod 的调度，K8S 提供下面四种调度方式：

- 自动调度(无特殊设置)
  完全有 Scheduler 经过一系列计算得出
- 定向调度(强制性)
  - 定向调度是强制性的，如果不满足则 pod 运行失败
  - NodeName 直接指定 nodeName
  - NodeSelector 指定满足 label 匹配条件的 node
- 亲和性调度
  - NodeAffinity 优先调度到指定 node，比如特定硬件资源
    - requiredDuringSchedulingIgnoreDuringExecution
      - **强制**调度到匹配的 node 节点，相比定向调度，支持更丰富的匹配模式
      - 可以同时设置多个条件，terms 间是 or 的关系；matchExpressions 间是 and 的关系
    - preferedDuringSchedulingIgnoreDuringExecution
      - **非强制** 匹配 node。
      - 可以设置多个匹配条件，每个条件可设置一个权重`weight`，根据权重匹配
  - PodAffinity 以已经运行的 Pod 作为参照，优先调度到亲和的 Pod 相同的 node。例如：频繁网络通信的 pod，可以减少网络损耗
    - 同样有"强制"和"非强制"两种匹配方式
  - PodAntiAffinity 与上面相反，避开符合条件的 Pod。例如：提供相同网络功能的 pod，可以通过反亲和性实现更好的负载均衡
- 污点(容忍)
  - Taints 在 Node 设置后，可以避免 Pod 分配到上面，或者驱逐 Pod。
  - Toleration 可以在 Pod 设置容忍，这样还是可以调度到有 Taint 的 Node 上。

### Node 亲和性

```yml
# pod.spec.affinity.nodeAffinity
requiredDuringSchedulingIgnoreDuringExecution <[]Object> # 必须匹配才调度，强制
  nodeSelectorTerms # 节点选择列表
    matchFields # 按节点字段选择
    matchExpressions  # 按节点标签选择
      key # 键
      value # 值
      operator  # 关系符： Exists, DoesNotExist, In, NotIn, Gt, Lt
```

- NodeAffinity 注意事项：
  - 如果同时定义了 nodeSelector 和 nodeAffinity，那么必须两个条件都得到满足，Pod 才能运行在指定的 Node 上
  - 如果 nodeAffinity 指定了多个 nodeSelectorTerms，那么只需要其中一个能够匹配成功即可
  - 如果一个 nodeSelectorTerms 中有多个 matchExpressions，则一个节点必须满足所有的才能匹配成功
  - 如果一个 Pod 所在的 Node 在 Pod 运行期间其标签发生了变化，不再符合该 Pod 的节点亲和性需求，则系统将**忽略**此变化

### Pod 亲和性

```yml
# explain pod.spec.affinity.podAffinity
requiredDuringSchedulingIgnoredDuringExecution <[]Object> # 硬限制
  namespaces  # 指定参照 pod 的 namespace
  topologyKey # 指定调度作用域
  labelSelector # 标签选择器
    matchExpressions  # 按 pod 标签列出的选择器列表
      key
      values
      operator
    matchLabels # 指多个 matchExpressions 映射的内容
preferredDuringSchedulingIgnoredDuringExecution <[]Object> # 软限制
  podAffinityTerm # 选项
    namespaces
    topologyKey
    labelSelector
      matchExpressions
        key
        values
        operator
      matchLabels
  weight  # 倾向权重，[1, 100]
```

- topologyKey(调度作用域)
  找到目标参照 Pod 后，根据目标 Pod 的哪些方面进行亲和性调度。如：
  - `kubernetes.io/hostname` 以 Node 节点为区分维度
  - `beta.kubernetes.io/os` 以 Node 节点的操作系统类型为区分维度

### taint

- taint 格式：`key=value:effect`，"key=value"是污点的标签，effect 描述污点的作用，有下面三项：

  - PreferNoSchedule: K8S 尽量避免把 Pod 调度到具有该污点的 Node 上，除非没有别的 Node 可调度
  - NoSchedule: K8S 将不会把 Pod 调度到具有该污点的 Node 上，但不会影响当前 Node 上已存在的 Pod
  - NoExecute: K8S 将不会把 Pod 调度到具有该污点的 Node 上，同时会将 Node 上已存在的 Pod 驱离

- K8S 的 master 节点默认带污点`node-role.kubernetes.io/master:NoSchedule`，所以一般不分配 Pod

### toleration

```yml
# explain pod.spec.tolerations
tolerations: # 可添加多个容忍项
  - key: "taintKey" # 要容忍的污点的 key
    operator: "Equal" # 操作符
    value: "taintValue" # 容忍的污点 value
    effect: "NoExecute" # 添加容忍的规则，这个规则必须和 node 标记的污点规则一致
```

# Network

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

# Controller

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
  strategy: # 镜像更新策略
    type: RollingUpdate # 滚动更新策略
    rollingUpdate:  # 滚动更新
      maxSurge: 30% # 更新过程中，最大额外可存在的副本数，可以为百分比或整数，默认 25%
      maxUnavailable: 30% # 最大不可用状态的 pod 数，可以为百分比或整数，默认 25%
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

- Deployment 自动创建的 RS 的 name 是在 Deployment name 后边加`-xxx`随机码。
  其他子资源也都是相同命名方法。

- revisionHistoryLimit
  保留历史版本以便进行版本回退，通过保留 RS 实现

- 调用`get`命令后各字段解释：

  - READY
    处于 READY 状态的 Pod 数量 / 总数量
  - UP-TO-DATE
    处于最新版本的 Pod 数量
  - AVAILABLE
    当前可用 Pod 数量

- 镜像更新策略

  - Recreate
    创建出新的 Pod 前会先杀掉所有已存在的 Pod
  - RollingUpdate(默认值)
    滚动更新，杀死一部分旧版本，启动一部分新版本

- 版本回退
  升级时会创建新的 ReplicaSet，但是旧的不会删除，以便版本回退。

- 金丝雀(Canary)发布(灰度发布)
  在 Deployment 设置了新版镜像后立刻暂停，这时候只有极少数的新版 Pod 启动，观察一段时间后，再决定是回滚还是继续升级。
  `kubectl set image deploy deploy-name nginx=nginx:1.17.4 -n dev && kubectl rollout pause deployment deploy-name -n dev`
  - 可以用`kubectl rollout status deploy deploy-name -n dev`查看状态
  - 如果新版 Pod 没有问题，则执行`kubectl rollout resume deploy deploy-name -n dev`继续
  - 如果新版有问题，则执行`kubectl rollout undo deploy deploy-name --to-revision=1 -n dev`回滚

## HPA

可以用 HPA(Horizontal Pod Autoscaler)控制器自动化、智能化的实现扩缩容，避免手工执行`kubectl scale`命令。
HPA 可以获取每个 Pod 的利用率，然后和 HPA 中定义的指标进行对比，同时计算出需要伸缩的具体值，最后实现 Pod 数量的调整。
HPA 通过操作 Deployment 实现功能。

```yml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-name
  namespace: dev
spec:
  minReplicas: 1 # 最小 pod 数量
  maxReplicas: 10 # 最大 pod 数量
  targetCPUUtilizationPercentage: 30 # CPU 使用率指标
  scaleTargetRef: # 制定要控制的 pod 信息，通过 Deployment 作用
    apiVersion: app/v1
    kind: Deployment
    name: nginx
```

- `get`命令结果
  - TARGETS
    显示实际 CPU 负载/设定 CPU 负载
  - REPLICAS
    实际的 Pod 数

```yml
// describe
```

- 扩容后，如果负载降低到阈值以下，会自动逐步缩容

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

- pod 名称示例：
  `nginx-web-0`
- pvc 名称示例：
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

# CRD(Custom Resource Definition)

CRD 是一种自定义资源，可以自己导入资源模板从而在 K8S 中创建自己的资源类型。
CRD 在 K8S 开源周边中应用很普遍，执行 `kubectl get crd` 就可以获取 CRD 列表，
比如前面说的**hpa**其实就是一种 CRD 资源。

CRD 要先定义再使用(创建资源)

```yml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # 名称必须与下面的spec字段匹配，格式为: <plural>.<group>
  name: crontabs.crd.test.com
spec:
  # 用于REST API的组名称: /apis/<group>/<version>
  group: crd.test.com
  versions:
    - name: v1
      # 每个版本都可以通过服务标志启用/禁用。
      served: true
      # 必须将一个且只有一个版本标记为存储版本。
      storage: true
  scope: Namespaced # 指定crd资源作用范围在命名空间或集群
  names:
    # URL中使用的复数名称: /apis/<group>/<version>/<plural>
    plural: crontabs
    # 在CLI(shell界面输入的参数)上用作别名并用于显示的单数名称
    singular: crontab
    kind: CronTab
    # 短名称允许短字符串匹配CLI上的资源，意识就是能通过kubectl 在查看资源的时候使用该资源的简名称来获取。
    shortNames:
      - ct
  validation: # 创建资源时，对资源中各字段的验证机制
  additionalPrinterColumns: # 定义在 `kubectl get xxx` 中以列表显示时，显示哪些列
    - name: Replicas
      type: integer
      JSONPath: .spec.replicas
      subresources:
  scale: # 定义 `kubectl scale` 时应该执行什么伸缩动作，如果没有定义则资源创建后无法伸缩
    specReplicasPath: .spec.replicas
    statusReplicasPath: .status.replicas
```
