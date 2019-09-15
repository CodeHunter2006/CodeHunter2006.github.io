---
layout: post
title:  "Docker中一些关键概念记录"
date:   2019-09-14 10:00:00 +0800
tags: Docker
---

# Docker不是虚拟机，只是一个文件层
之前把Docker理解为一个无界面的虚拟机，因为没有界面所以速度较快，这种理解比较粗浅。下面才是Docker真正的原理：

在Linux中有个概念"一切皆文件"，即程序与外界的一切操作都被抽象为文件操作，每个输入、输出都可以以FD(Filde Descriptor，文件描述符)来作为操作句柄。既然都是文件，那么可以用一个文件层来包裹程序，对程序的各种输入、输出进行反馈即可，让程序觉得自己是在一个完整的电脑上运行的。例如程序想读取显卡信息，不需要有真的显卡，只要在文件层立刻返回某个设置好的值就可以。

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_2.jpg)

Docker就提供了这样一个文件层，让程序以为自己运行在一个操作系统环境中。多个文件层可以嵌套，而每个文件层只要记录自己和基础层之间的差异就可以。这些文件层都是只读的，如果需要改变某些内容，就在最接近程序的地方增加一个写入层(Container)，保存与程序相关的内容。


![Docker VS VM](/assets/images/2019-09-14-Docker_basic_1.jpg)

**Docker与虚拟机相比:**

| 比较项\容器类型 | Docker容器 | 虚拟机（VM）|
| -------------- | --------- | ----------- |
|操作系统|与宿主机共享OS | 宿主机OS上运行虚拟机OS|
|存储大小|硬件小，便于存储与传输|镜像庞大（vmdk,vdi等）|
|运行性能|几乎无额外性能损失|操作系统额外的CPU,内存消耗|
|移植性|轻便，灵活，适用于Linux|笨重，与虚拟机耦合度高|
|硬件亲和性|面向软件开发者|面向硬件运维者|
|部署速度|快速，秒级|较慢，分钟级|

* PS: 虽然Docker提供了写入层，但是通常我们只写一些临时内容，因为这个写入层只保留在Container中，一旦Container删除则数据就是。通常通过Volume将主要数据输出到宿主机或某个网络地址上。


# Docker镜像的增量构建
Docker镜像可以与Git控制下的项目相类比，每个镜像相当于一个git库，镜像上的每一层相当于一个git commit，只要保存自己变更的那部分就可以了。所以镜像的构建是增量式的。

比如基础镜像是某linux版本A(100M)，python环境镜像基于A构建了B(110M)，scrapy环境镜像基于B构建了C(111M)。<br/>
这时总容量关系是 A(100M) < B(110M) < C(111M)，每个层的容量关系是 A(100M) > B(10M) > C(1M)。<br/>
如果这时需要pull D，D是基于B的总容量是112M，那么实际的流量和容量只有2M的差异部分，其他都利用了现有镜像。

这种增量构建有三个好处：
1. 每次只需要关注变更点，容易维护。
2. 本地保存大量镜像，实际上公共部分只保留一份，节省容量。类似dll的机制
3. 网络传输时，只需要下载差异部分，节省流量。

## 关于增量构建的一些技巧

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_4.jpg)

### Base操作系统镜像
操作系统镜像往往是镜像的Base，所以选择一个合适的系统镜像可以降低总的容量，但是过于精简的又缺少足够的功能。`Alpine`(阿尔卑斯山)做为最精简的linux收到Docker社区的欢迎，几乎所有官方Docker镜像都提供了Alpine版本的Tag。


![Docker VS VM](/assets/images/2019-09-14-Docker_basic_3.jpg)

### DockerHub + GitHub
把DockerFile放在GitHub，可以直接对系统环境进行增量管理。DockerHub对GitHub很友好可以直接集成GitHub账号和库。

![Docker VS VM](/assets/images/2019-09-14-Docker_basic_5.jpg)

### Dll Hell问题
在利用DockerHub的镜像时，要注意版本号、Tags。例如Python官方Docker页面，有如下版本：
```
3.7.4-alpine3.10, 3.7-alpine3.10, 3-alpine3.10, alpine3.10, 3.7.4-alpine, 3.7-alpine, 3-alpine, alpine
```
这些Tags其实都表示了**当前的**一个同一个版本。比如你现在执行`docker pull python:alpine`和`docker pull python:3.7.4-alpine3.10`下载的镜像是完全一致的。<br/>
但是如果一个月后发布了`3.7.4-alpine3.11`，这时执行`docker pull python:alpine`会自动下载最新的版本而不是一个月前的`3.7.4-alpine3.10`版本，这就产生了版本差异。而这种版本差异可能会导致Dll Hell问题(由于dll文件版本升级导致的深坑)。<br/>
为了避免Dell Hell，我们最好**每次都指定最精确的版本号**。