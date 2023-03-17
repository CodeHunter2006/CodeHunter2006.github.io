---
layout: post
title: "VSCode+Docker+Debian+Python remote debug"
date: 2023-03-17 22:00:00 +0800
tags: Python Docker VSCode
---

`Dockerfile`

```dockerfile
FROM python:3.7.16-bullseye

# 更新软件源
RUN sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list; sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list;
# 安装 ssh server
RUN apt-get update && apt-get install -y openssh-server

# 添加非 root 用户
RUN useradd -ms /bin/bash username
# 设置密码
RUN echo 'username:password' | chpasswd
# 加入 su 组
RUN usermod -aG sudo username

# 这里用于开启 root 登录权限，未必能成功
# RUN echo 'root:PASSWORD' | chpasswd
# RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

# 创建必要的文件夹
RUN mkdir /var/run/sshd

# 暴露 ssh 端口
EXPOSE 22

# 启动 ssh server
CMD ["/usr/sbin/sshd", "-D"]
```

`docker build -f Dockerfile -t debug_test .`

`docker run -d -P -v /dockershare:/home/username/share --name debug_test debug_test`
