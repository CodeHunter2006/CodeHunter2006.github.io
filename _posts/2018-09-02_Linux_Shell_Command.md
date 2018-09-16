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

反向显示列表内容
ll -t | tac

以分页形式查看命令结果
xxx |less

当前路径打开终端
ctrl+shift+t

复制 ctrl+insert
粘贴 shift+insert

查询命令帮助
man xxx

当前文件夹正则搜索
find -name "xxx"

全盘定位文件
locate xxx
locate -r xxx 正则查找

查找特定时间之后创建或修改的文件
touch -t 201407201710.00 abc    //在当前文件夹创建一个 2014年07月20日17点10分00秒 创建的临时文件
find -newer abc    // 在当前文件夹下，查找比刚才那个临时文件更新的文件

更新locate数据库
sudo updatedb

按文件内容搜索
grep "test" -r /usr

连续执行命令
xxx;xxx

管道，前面命令结果作为后面命令参数
xxxx | xxxx

whereis xxx
查找命令路径

which xxx
判断xxx的类型，是bin、函数还是gems等

cat xxx | grep error
查找某文件中包含error字符串的行

cat /etc/issue
查看操作系统版本

ifconfig
查看IP信息
