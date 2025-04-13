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

- `docker pull repository_name/image_name:tag`
  运行一个镜像为容器
  - ":tag"为可选项，默认为 latest
  - `--platform arch` 默认情况下会根据当前系统架构自动选择，可选项：`linux/arm64`、`linux/amd64`等

# run

- `docker run xxx`
  运行某个镜像

- `--rm`容器退出时自动销毁，可用于临时测试
- `-P` Publish all exposed ports to random ports，将容器内的端口暴漏为外部主机的随机端口
- `-p outterPort:innerPort` 将容器内的端口暴漏为外部主机的指定端口，可以设置多个
- `-v outterPath:innerPath` 映射内外文件夹，可以设置多个
- `--name containerName` 为容器指定名称

# start

- `docker start -i xxx`
  - 以`docker run -it ...`启动后，退出 terminal 时容器也停止了，用这个命令可以继续运行

# stop

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

- `docker inspect containerName`
  查看一个容器的详细信息，比如 ip 地址

- `docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' containerName`
  查看容器的 ip 地址

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

```bash
# 删除一系列，"<none>:<none>" 镜像
docker rmi $(docker images -f "dangling=true" -q)
```

导出镜像文件

```
docker save -o file.tar xx:xx
```

导入镜像文件

```
docker load < file.tar
```

`docker logs container_name`
查看标准输出日志

`docker commit container_name image_name`
将 container 导出为镜像

- `docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]`
  重命名本地镜像，并且将其归入某一仓库

- `docker push REGISTRYHOST:5000/USERNAME/NAME:latest`
  将镜像推送到远端仓库

  - `REGISTRYHOST`是仓库 host，可以是 dockerhub 或自建仓库
  - `USERNAME` 是仓库中的账号名
  - `NAME` 是镜像名
  - `latest` 是版本 tag

- `docker pull [REGISTRYHOST:5000/]USERNAME/NAME:latest`
  拉取镜像到本地。如果不设`REGISTRYHOST`则使用默认仓库。

`docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`
在运行的容器中执行命令

- `-d` 分离模式，在后台运行
- `-i` 即使没有附加也保持 STDIN 打开
- `-t` 分配一个伪终端

- `docker build -f dockerFile -t imageName .`
  通过指定的`dockerFile` buid 出指定镜像文件到`.`

- `docker search imageName`
  搜索 docker hub 中是否有镜像

- `docker login [私有镜像库地址]`
  登录到一个镜像库，如果不指定镜像库，则默认登录官方镜像库(docker-hub)

  - `-u userName`
  - `-p password`

- `docker logout`
  登出镜像库，如果不指定，则默认登出 docker-hub。

- `doccker system`
  查看各种系统信息
  - `df` 磁盘占用情况
