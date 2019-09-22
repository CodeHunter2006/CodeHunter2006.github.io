---
layout: post
title:  "Airtest介绍"
date:   2019-09-08 10:00:00 +0800
tags: UIAutomationTest CI
---
<video id="video" controls="controls" style="max-width: 900px;" preload="none" poster="/assets/images/20190908_Airtest_Introduction_1.png">
<source id="mp4" src="/assets/videos/20190908_Airtest_Introduction_1.mp4" type="video/mp4">
</video><br/>
Airtest是网易游戏18年1月份推出的一款开源的、跨平台的UI自动化测试IDE。上面是网易游戏QA技术总监刘欣在18年Google技术分享会上的演讲视频节选，显示了多台手机在Airtest+Firebase框架下并行执行自动化测试。<br/>
Airtest受到了谷歌的推荐，在网易游戏内部使用了三年，到现在已经开源了一年多，具有很优点，值得学习和参考。

# 什么是UIAutomationTest(UI自动化测试)？
![example](/assets/images/20190908_Airtest_Introduction_2.gif)<br/>
用程序模拟用户对画面的对象识别、按钮点击、文本输入、结果判断，来进行测试。一般情况下，产品的最终形态往往要给用户提供一套UI界面来操作，所以UI自动化测试非常适合做冒烟测试，验证整个系统的基本Case是否有效、可用。

# UIAutomationTest的难点是什么，Airtest有什么改进？
UI自动化测试最大的难点在于画面上对象的识别。

### 旧的方式
以往是通过截图的方式解决：在编写Case时对想要识别或点击的图片截图，然后在Case运行时对图片进行比较，如果一致则命中。
* 这种形式很容易发生一个像素的偏差导致无法命中，可以设定一个模糊度，对一定范围内的像素点做比较，提高命中率。但是在分辨率变化、少量图片纹理变化的情况下还是容易无法命中，导致后期Case维护困难、甚至放弃自动化测试。

![OpenCV](/assets/images/20190908_Airtest_Introduction_1.jpg)
### 采用OpenCV识别
对于图片，Airtest利用较为成熟的OpenCV来记录编写Case时的图片的特征，这些特征包括了一张图片内容的方方面面，例如尖角、圆边、线条、明暗区域关系...然后在测试时在屏幕各处寻找特征值综合结果最为匹配的对象。
* 如果发生了分辨率变化，有些特征值可能改变，但是很多特征值不会改变，这样就避免了以前分辨率稍有变化就重写Case的问题。
* 如果纹理变化不大，多数特征值的计算结果仍然与之前的特征值相同或接近，可以通过设定合理的匹配门槛(threshold)来提高命中率，同样减少了重写Case的麻烦。
* 在识别特征前，通常都会把图片先处理为灰度图，这样也避免了颜色变化导致的命中失败问题。如果想将颜色通道作为匹配特征，可以设定`RGB = True`。
* 虽然采用OpenCV，但是某些情况下仍然有问题，例如较小的文字字体发生变化时，特征值将发生剧烈变化，从而导致无法命中。为了应对这个问题，Airtest在识别图片的同时也支持控件树结构的操作。

### 通过POCO直接查找控件树结构
POCO是一套UI控件搜索的自动化框架，支持Android、IOS这样的操作系统，也支持Unity3D、Cocos2dx-js这样的游戏引擎，支持Chorme浏览器、WeChat Applet这种Web平台，同时也在开发Windows、MacOS这些尚未支持的系统。Airtest中整合了开源的POCO，可以调用各平台的POCO-SDK精确搜索控件进行识别和点击等操作。

### Airtest的其他改进点
* Airtest基于Python，可以利用大量Python库实现辅助功能，也可以整合到Python脚本中作为CI(Continuous Integration持续集成)的回归测试部分。
* AirtestIDE整合了常用的UI测试功能，可以方便的在IDE中编写Case。

### Airtest的发展方向
![structure](/assets/images/20190908_Airtest_Introduction_3.jpg)<br/>


# Airtest一些用法记录

门槛
严格门槛
RGB通道
自定义分辨率变化函数
图片识别工具的使用
.air间引用
与Python结合

# 参考资料：

官方文档：
http://airtest.netease.com/docs/docs_AirtestIDE-zh_CN/index.html

演讲视频：
https://www.youtube.com/watch?v=lJGBDFgOulY

开源地址：
https://github.com/AirtestProject/Airtest

(参考)开源访谈：
https://www.oschina.net/question/3820517_2278684?nocache=1524704966955


扩展思路：
用腾讯云API实现文本的识别和坐标定位，用来解决字体变化问题。也可以用tesseract实现自己的OCR应用。
但是这种识别的局限也比较大，只有文字可以识别，另外也有准确率问题。
可以增加字体，增加识别率。可以增加特定的训练集，做到识别率趋于100%，但是也需要辅助位置识别，先识别文本所在位置。
整体投入收益比不足，还不如使用现成的OpenCV的方案，只实现最基本的Case。
任何时候都要先测算一下投入收益比，否则很容易导致投入过大收益小。

PS:<br/>
多年前用Visualstudio的CodedUI(类似POCO)做过自动化测试提高效率，虽然最终实现了测试目标，但是有很多局限性，比如必须是在Windows上原生的Windows窗体程序才可以、必须安装容量巨大的Visualstudio、无法支持浏览器等，实现起来很不方便。接触到Airtest后感受到技术的进步(OpenCV)和开源社区的强大(POCO)，希望能利用好Airtest，作为鼠标键盘精灵和CodedUI很好的替代(与Python良好结合)。
