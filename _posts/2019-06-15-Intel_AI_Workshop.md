---
layout: post
title:  "Intel AI Workshop 笔记"
date:   2019-06-15 22:00:00 +0800
tags: CNN
---
![Intel_AI_Workshop.jpg](/assets/201906151330_Intel_AI_Workshop.jpg)

今天参加了"Intel AI Workshop - From Data Center to Edge – An optimized path using Intel Architecture"。

通过几位老师(Gu Jianjun、Cao Huiyan、Pei Fanjiang 和他们的老大Derek Fattibene)的教学，我把神经网络的数据整理、模型训练和模型推理走了一个完整的流程。正好借这个机会，记录一个卷积神经网络的实际案例，之后陆续把课程笔记写下来。

### 课程题目
我们想通过图像识别(视频)对车辆的生产商、型号、生产年代进行识别

使用VMMRdb(Vehicle Make and Model Recognition Dataset)作为数据集

利用Tensor Flow平台进行训练

最终利用Intel OpenVINO平台对模型进行部署和推理<br/>
( __推理__ 就是指把模型部署到终端，对数据进行计算得出结论的过程，也就是模型的应用)

### 对数据探索分析


### 训练模型


### 模型分析


### 边缘部署/推理


### AI发展状况介绍，Intel提供哪些资源（广告较多，放在最后）
* 为什么要利用AI？

Data deluge(数据洪水)问题，每天产生的数据量：人产生25GB、智能汽车50GB、医院3TB、飞机40TB、智能工厂1PB...。这么多数据用人工只能达到很有限的广度和深度，最终只能靠AI。

* 什么是AI？

AI包含Machine Learning包含Deep Learning。一般都要经过训练(training)生成模型(model)最后进行推理(inference)。

* hardware(硬广)

Intel提供CPU、集成GPU和计算棒或PCI计算卡。

绝大多数的业务使用CPU进行计算，但是随着计算量的增加，需要专用硬件进行计算。

* 软件

Intel提供一个工具集OpenVINO，可以将TensorFlow/Caffe2等框架计算好的模型进行部署，针对Intel的硬件生成优化代码，最终高效的进行推导。

* 社区

Intel提供一系列学习资源，支持AI社区的发展<br/>
[https://software.intel.com/ai-academy](https://software.intel.com/ai-academy)<br/>
[https://software.intel.com/ai/courses](https://software.intel.com/ai/courses)<br/>
[https://software.intel.com/ai-academy/tools/devcloud](https://software.intel.com/ai-academy/tools/devcloud)<br/>
[https://communities.intel.com/community/tech/intel-ai-academy](https://communities.intel.com/community/tech/intel-ai-academy)<br/>
[https://devmesh.intel.com](https://devmesh.intel.com)



![Intel_AI_Workshop.jpg](/assets/201906151330_Intel_AI_Workshop_2.jpg)
会后讲师合影（左起：Derek Fattibene、Cao Huiyan、Pei Fanjiang、Gu Jianjun）

[课程对应的PDF文档链接](https://software.intel.com/en-us/ai/courses)
