---
layout: post
title:  "Linux 常用命令"
date:   2018-09-02 10:00:00 +0800
tags: Linux
---

Ctrl+C 打断程序运行
Ctrl+Z 暂停进程
jobs 查看已经暂定的进程
fg ### 将后台进程加载到前台运行
bg ### 在后台继续运行暂停的进程
如果要退出terminal时仍然运行后台进程，进程的输出要重定向，例如 xxx >>tmp.log

查看系统版本
cat /etc/issue

改变文件权限为可执行可修改
chmod 755 ./xxx

更新一下文件的最新修改时间
touch ./xxx

将某文件或文件夹授权给某用户
chown username filefoldername


insmod
将文件安装到内核
通常是驱动。

ls 列出文件名
ls -a 列出所有文件，包含隐藏文件
ls -l 列出详细信息，最后会表示软连接信息

ll
列出详细信息
显示的时间为文件或文件夹的修改时间

du --apparent-size
显示当前文件夹空间以及各子文件夹空间

pwd
当前目录
