---
layout: post
title: "Docker build CMD"
date: 2019-01-05 09:00:00 +0800
tags: Docker
---

Docker 镜像可以通过在容器中进行命令操作创建，但更好的方式是通过 DockerFile 进行 Build。这样把 DockerFile 用放到 GitHub，版本控制、易于分享、可以实现自动化 Build。

下面就列出一些常用 DockerFile 命令

### FROM

设定此次编译的目标镜像的基础镜像

`FROM [--platform=<platform>] <image> [AS <name>]`

- `FROM alpine:3.8`
  以`alpine:3.8`为当前基础镜像进行操作

- `FROM aaa as bbb`
  这里的 bbb 是某个中间镜像，之后可以被 copy 调用把中间生成文件拷走

- `--platform=arch` 可以指定架构类型，如`linux/arm64`

### MAINTAINER

指明镜像制作者的名称及电子邮件，以便之后的使用者联系。

```
MAINTAINER codehunter2006 "codehunter2006@gmail.com"
```

### RUN

执行一些命令，用`\`换行

```
RUN echo 'hello world!' \
    > /home/user1/file1.txt
```

### COPY

将 DockerFile 所在目录下的文件复制到容器中，会在容器中自动创建相应文件夹

### ADD

ADD 和 COPY 功能基本相同，COPY 相比更简易常用。

- ADD 独有的两个功能：
  - 解压压缩文件并把它们添加到镜像中
  - 从 url 拷贝文件到镜像中

### EXPOSE

指定容器向外暴露的端口号

```
EXPOSE 8080
EXPOSE 9090
```

### WORKDIR

指定 Build 过程中在容器中执行命令时的工作目录

```
WORKDIR /home/
WORKDIR user1
RUN echo 'hello world!' > file1.txt
```

上面的命令最终创建了文件`/home/user1/file1.txt`

### ONBUILD

目标镜像被其他人做基础镜像进行编译时执行的命令

### USER

指定镜像运行时的用户

```
USER apache
```

### VOLUME

指定主机目录或文件映射到容器的目录或文件

```
VOLUME /data/dbout /data/dbin
```

### CMD

指定容器启动时要执行的命令。可以被 docker run 命令的参数覆盖。

### ENTRYPOINT

和 CMD 类似，指定容器启动时要执行的命令。如果 docker run 也有命令，会被放在 ENTRYPOINT 后面。

- 在`docker run`命令中可以通过`--entrypoint xxx`覆盖`ENTRYPOINT`。

### ARG

在 docker build 时可以通过`--build-arg <参数名>=<值>`来覆盖。

- `ARG <参数名>[=<默认值>]`

### LABEL

为镜像添加一些元数据

- `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
