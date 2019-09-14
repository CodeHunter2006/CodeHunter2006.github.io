---
layout: post
title:  "Python笔记——与C++和Ruby对比学习"
date:   2019-09-14 10:00:00 +0800
tags: Python
---

# 基本语法

### 变量作用域
全局变量、局部变量 global



# pip

`pip freeze > requirements.txt`
将当前Python环境依赖的外部库列出，生成`requirements.txt`文件，类似Ruby的`Gem.lock`精确的记录了依赖版本。该文件可以在其他机器上使用，以便批量安装依赖包。

`pip install -r requirements.txt`
按照`requirements.txt`列表中记录的库的版本，安装所有包。
