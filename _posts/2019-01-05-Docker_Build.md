---
layout: post
title:  "Docker build CMD"
date:   2019-01-05 09:00:00 +0800
tags: Docker
---

Docker镜像可以通过在容器中进行命令操作创建，但更好的方式是通过DockerFile进行Build。这样把DockerFile用放到GitHub，版本控制、易于分享、可以实现自动化Build。

下面就列出一些常用DockerFile命令

### FROM
设定此次编译的目标镜像的基础镜像
```
FROM alpine:3.8
```

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
将DockerFile所在目录下的文件复制到容器中，会在容器中自动创建相应文件夹

### EXPOSE
指定容器向外暴露的端口号
```
EXPOSE 8080 
EXPOSE 9090
```

### WORKDIR
指定Build过程中在容器中执行命令时的工作目录
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
指定容器启动时要执行的命令。可以被docker run命令的参数覆盖。

### ENTRYPOINT
和CMD类似，指定容器启动时要执行的命令。如果docker run也有命令，会被放在ENTRYPOINT后面。

* 在`docker run`命令中可以通过`--entrypoint xxx`覆盖`ENTRYPOINT`。

