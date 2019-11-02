---
layout: post
title: "PlantUML介绍"
date: 2019-11-02 21:00:00 +0800
tags: UML Docker VSCode
---

![PlantUML](/assets/images/2019-11-02-PlantUML_introduce_1.jpg)
PlantUML 是一套从文本生成(plant) UML 图的开源系统。它的语法简单、图形惊艳、社区活跃、插件丰富，非常适合程序开发制图，在 VSCode 插件配合下可以方便的将代码转换为 UML 图形以便快速分析。

# 特点

- 以类似 Markdown 的语法表示逻辑关系，具有较好的可读性，也较容易制作相关的分析和生成工具、插件。
- 以文本格式存储，避免了以前各种工具制作 UML 后不容易版本控制的问题，可以直接用 Git 等 VCS 管理；另外，这种显式的格式也方便各种工具间互转。
- 支持所有 UML 视图类型，还支持常用的思维导图等图形，非常符合软件开发、管理的需要。
- 生成的矢量图可以任意缩放，不会有分辨率问题，清晰、美观。
- 社区活跃，有各种语言的转换插件，例如可以从`.go`转换为`.plantuml`，然后可以通过类图快速分析项目结构。
- 强大、易用的渲染引擎，可以方便的生成高清图片。为了防止代码外泄，可以利用官方提供的 Docker 在本地快速搭建渲染引擎。

# 图形及语法示例

PlantUML 支持类图、时序图、活动图、对象图等所有 UML 图形。同时还支持思维导图、甘特图、WBS(Work Breakdown Structure)图等软件开发常用图形。

首先要用对应的语法写一个文本文件，以类图为例：

```plantuml
ClassA <|-- ClassB
ClassA <-- ClassC
```

将自动生成矢量图：
![ClassDiagram](/assets/images/2019-11-02-PlantUML_introduce_2.png)

# VSCode 插件安装(mac)

PlantUML 插件可以方便的进行文本到矢量图转换，并且提供了 markdown 内嵌预览 PlantUML 功能。

在 VSCode 插件中搜索 PlantUML，即可安装插件。

之后需要按照插件说明中的命令安装依赖的 java 和 graphviz。

```
brew cask install java
brew install graphviz
```

在`.plantuml`/`.puml`等格式的文本文件中，按`ALT+d`(windows)或`OPT+d`(mac)，可以生成矢量图形的预览窗口。

在 markdown 中，如果代码格式标记为`plantuml`，则该代码的 markdown 预览会被自动替换为对应的 UML 图片。

设置了`plantuml.exportOutDir`配置项后，可以在该文件夹下生成导出的图片。

```
"plantuml.diagramsRoot": "docs/diagrams/src",
"plantuml.exportOutDir": "docs/diagrams/out"
```

# 本地 Docker 启动渲染服务

PlantUML 的图形渲染需要一个渲染服务，官方虽然提供了免费的渲染 http 接口，但是为了保护公司代码，需要我们自己在局域网启动一个渲染服务。

下载并启动 Docker：

```bash
docker run -d -p 8080:8080 plantuml/plantuml-server:tomcat
```

设置 VSCode 配置项：

```
"plantuml.server": "http://127.0.0.1:8080",
"plantuml.render": "PlantUMLServer",
```

之后生成预览和导出图片就会使用本地的渲染服务了。

# 参考链接

- [PlantUML 官网](http://plantuml.com/index)
- [PlantUML 开源地址](https://github.com/plantuml/plantuml)
- [VSCode 插件开源地址](https://github.com/qjebbs/vscode-plantuml)
- [Go 语言插件开源地址](https://github.com/jfeliu007/goplantuml)
- [渲染服务 DockerHub 地址](https://hub.docker.com/r/plantuml/plantuml-server)
