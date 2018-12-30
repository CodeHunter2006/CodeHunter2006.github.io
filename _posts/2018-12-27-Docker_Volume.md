---
layout: post
title:  "Docker volume"
date:   2018-12-27 09:00:00 +0800
tags: Docker VirtualBox
---
Docker容器进行文件写入时，会写到容器层，当容器删除后，文件也就被删除了。

Docker采用volume(卷)的形式向容器提供持久化的存储。

Docker要想实现文件写入不被删除，有下面几种方法：

* 不使用volume，利用docker commit命令，把当前容器持久化为一个新的镜像。

* 在docker run时，通过"-v 容器目录"的形式挂载一个主机上的目录或文件

* 利用volume container，创建一个挂载了volume的容器，其他容器可以通过"--volumes-from"命令继承这个容器的写入空间。

* 利用"docker volume"命令创建专门的volume对象，可以在容器启动时挂载。注意要清理不用的volume。

### 在Windows下在VirtualBox中运行Docker+volume的坑
通常情况，多个容器共用一个volume时，会有一个权限问题。比如A容器中用户A1写了一个文件F1，当B容器的用户B1修改这个文件时，就会权限不足。

解决方法有两种：
* 1.在宿主中建立各个角色，UID要与容器中一致，然后修改文件的拥有者。
* 2.各个容器都是用root权限，这样就不会有权限问题了。这种方案安全性方面有点问题。

在Window操作系统中用VirtualBox运行Linux(Alpine)虚拟机，通过共享文件夹Linux可以访问Windows的文件夹。但是在Linux中查看，发现权限永远是root的，及时通过chown也无法改变。所以上面第一种方案就没法实现了，只能用第2种方案解决。




