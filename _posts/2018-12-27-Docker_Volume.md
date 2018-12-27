---
layout: post
title:  "Docker volume"
date:   2018-12-27 09:00:00 +0800
tags: Docker
---
Docker容器进行文件写入时，会写到容器层，当容器删除后，文件也就被删除了。

Docker采用volume(卷)的形式向容器提供持久化的存储。

Docker要想实现文件写入不被删除，有下面几种方法：

* 不使用volume，利用docker commit命令，把当前容器持久化为一个新的镜像。

* 在docker run时，通过"-v 容器目录"的形式挂载一个主机上的目录或文件

* 利用volume container，创建一个挂载了volume的容器，其他容器可以通过"--volumes-from"命令继承这个容器的写入空间。

* 利用"docker volume"命令创建专门的volume对象，可以在容器启动时挂载。注意要清理不用的volume。
