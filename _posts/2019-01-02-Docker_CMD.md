---
layout: post
title: "一些Docker基本概念和常用命令"
date: 2019-01-02 09:00:00 +0800
tags: Docker
---

在 Docker 中可操作的元素有三种：Image(镜像)、Container(容器)、Volume(卷)

镜像和容器的差别就是是否运行，镜像是一种静态的，容器是一种正在运行中或是准备好运行参数的对象。卷是为了互联文件创建的对象。

Docker 和虚拟机原理不同，并不是完整的把进程执行在期内部，而是用一层写入层将进程的修改内容记录下来，所以容器删除后对应的写入数据也会丢失。为了保存写入的数据，就要用到卷。

※运行容器时，可以通过宿主系统的 ps 或 top 命令查看，在容器中执行的进程也可以在宿主中看到，也可以说明 Docker 并不是虚拟机。

创建一个容器时，会自动生成一个 **容器 ID** ，之后可以利用这个 ID 进行操作，不用全部输入，只要输入一部分 ID 可区分即可。

如果没有指定容器名称，会自动生成一个 **容器名称** ，一般是"形容词+名字"的形式，便于记忆。需要 ID 的位置输入名字也可以。

想要手动操作修改镜像，可以在启动时使用命令

```
docker run -it xxx/xxx:xxx bash
```

这样进入 Linux 操作系统时就会进入 shell，`-i`表示交互、`-t`表示终端，这样就有了交互终端。做完修改之后，如果想保存，可以利用`docker commit`命令保存为镜像。

在容器的终端内退出容器

```
Ctrl+D
```

在容器的终端内，将容器切换到后台运行

```
Ctrl+P+Q
```

之后可以通过`docker attach 容器标识`命令切换入容器终端内，注意往往进入后看起来没反应，只是没有输出。

一般可以通过 Build DockerFile 来编译出一个镜像，也可以通过容器创建。实际上，DockerFile 中的命令就好像在容器中执行了一系列操作一样，只是更容易读懂、避免了中间安装过程的时间和输出的信息造成扰乱，结合 Git 的话，就可以像维护代码那样来维护容器。

可以充分利用 Docker Hub，上面有各种官方 Docker 以及其他人制作的 Docekr。利用 Git Hub 可以维护 DockerFile，结合使用，随时可以组合出自己的容器集群配置方案。

下载一个镜像

```
// ":tag"为可选项
docker pull repository_name/image_name:tag
```

运行一个镜像为容器

```
docker run ...
```

优雅的(等待容器一段时间)停止一个容器

```
docker stop ...
```

查看当前运行的容器

```
docker ps
```

查看所有容器，包括未运行的

```
docker ps -a
```

查看一个容器的详细信息

```
docker inspect ...
```

查看当前所有镜像

```
docker images
```

删除一个容器

```
docker rm ...
```

`docker container rename srcName tarName` 重命名 container

删除一个镜像

```
docker rmi ...
```

※如果镜像之间有依赖关系，那么必须先删除子镜像才能删除根镜像

导出镜像文件

```
docker save ...
```

导入镜像文件

```
docker load ...
```

`docker commit container_name image_name`
将 container 导出为镜像

`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`
在运行的容器中执行命令

- `-d` 分离模式，在后台运行
- `-i` 即使没有附加也保持 STDIN 打开
- `-t` 分配一个伪终端
