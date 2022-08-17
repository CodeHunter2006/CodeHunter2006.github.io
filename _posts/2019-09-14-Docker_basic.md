---
layout: post
title: "Docker中一些关键概念记录"
date: 2019-09-14 10:00:00 +0800
tags: Docker
---

# Docker 不是虚拟机，只是一个文件层

之前把 Docker 理解为一个无界面的虚拟机，因为没有界面所以速度较快，这种理解比较粗浅。下面才是 Docker 真正的原理：

在 Linux 中有个概念"一切皆文件"，即程序与外界的一切操作都被抽象为文件操作，每个输入、输出都可以以 FD(Filde Descriptor，文件描述符)来作为操作句柄。既然都是文件，那么可以用一个文件层来包裹程序，对程序的各种输入、输出进行反馈即可，让程序觉得自己是在一个完整的电脑上运行的。例如程序想读取显卡信息，不需要有真的显卡，只要在文件层立刻返回某个设置好的值就可以。

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_2.jpg)

Docker 就提供了这样一个文件层，让程序以为自己运行在一个操作系统环境中。多个文件层可以嵌套，而每个文件层只要记录自己和基础层之间的差异就可以。这些文件层都是只读的，如果需要改变某些内容，就在最接近程序的地方增加一个写入层(Container)，保存与程序相关的内容。

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_1.jpg)

**Docker 与虚拟机相比:**

| 比较项\容器类型 | Docker 容器              | 虚拟机（VM）                |
| --------------- | ------------------------ | --------------------------- |
| 操作系统        | 与宿主机共享 OS          | 宿主机 OS 上运行虚拟机 OS   |
| 存储大小        | 硬件小，便于存储与传输   | 镜像庞大（vmdk,vdi 等）     |
| 运行性能        | 几乎无额外性能损失       | 操作系统额外的 CPU,内存消耗 |
| 移植性          | 轻便，灵活，适用于 Linux | 笨重，与虚拟机耦合度高      |
| 硬件亲和性      | 面向软件开发者           | 面向硬件运维者              |
| 部署速度        | 快速，秒级               | 较慢，分钟级                |

- PS: 虽然 Docker 提供了写入层，但是通常我们只写一些临时内容，因为这个写入层只保留在 Container 中，一旦 Container 删除则数据就是。通常通过 Volume 将主要数据输出到宿主机或某个网络地址上。

# Docker 镜像的增量构建

Docker 镜像可以与 Git 控制下的项目相类比，每个镜像相当于一个 git 库，镜像上的每一层相当于一个 git commit，只要保存自己变更的那部分就可以了。所以镜像的构建是增量式的。

比如基础镜像是某 linux 版本 A(100M)，python 环境镜像基于 A 构建了 B(110M)，scrapy 环境镜像基于 B 构建了 C(111M)。<br/>
这时总容量关系是 A(100M) < B(110M) < C(111M)，每个层的容量关系是 A(100M) > B(10M) > C(1M)。<br/>
如果这时需要 pull D，D 是基于 B 的总容量是 112M，那么实际的流量和容量只有 2M 的差异部分，其他都利用了现有镜像。

这种增量构建有三个好处：

1. 每次只需要关注变更点，容易维护。
2. 本地保存大量镜像，实际上公共部分只保留一份，节省容量。类似 dll 的机制
3. 网络传输时，只需要下载差异部分，节省流量。

## 关于增量构建的一些技巧

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_4.jpg)

### Base 操作系统镜像

操作系统镜像往往是镜像的 Base，所以选择一个合适的系统镜像可以降低总的容量，但是过于精简的又缺少足够的功能。`Alpine`(阿尔卑斯山)做为最精简的 linux 受到 Docker 社区的欢迎，几乎所有官方 Docker 镜像都提供了 Alpine 版本的 Tag。

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_3.jpg)

### DockerHub + GitHub

把 DockerFile 放在 GitHub，可以直接对系统环境进行增量管理。DockerHub 对 GitHub 很友好可以直接集成 GitHub 账号和库。

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_5.jpg)

### Dll Hell 问题

在利用 DockerHub 的镜像时，要注意版本号、Tags。例如 Python 官方 Docker 页面，有如下版本：

```
3.7.4-alpine3.10, 3.7-alpine3.10, 3-alpine3.10, alpine3.10, 3.7.4-alpine, 3.7-alpine, 3-alpine, alpine
```

这些 Tags 其实都表示了**当前的**一个同一个版本。比如你现在执行`docker pull python:alpine`和`docker pull python:3.7.4-alpine3.10`下载的镜像是完全一致的。<br/>
但是如果一个月后发布了`3.7.4-alpine3.11`，这时执行`docker pull python:alpine`会自动下载最新的版本而不是一个月前的`3.7.4-alpine3.10`版本，这就产生了版本差异。而这种版本差异可能会导致 Dll Hell 问题(由于 dll 文件版本升级导致的深坑)。<br/>
为了避免 Dell Hell，我们最好**每次都指定最精确的版本号**。

### 恢复 docker desktop

有时由于配置 docker desktop 使得 docker 无法启动，可以修改`~/.docker/daemon.json`解决。
