---
layout: post
title:  "Airtest介绍"
date:   2019-09-08 10:00:00 +0800
tags: UIAutomationTest CI
---
<video id="video" controls="controls" style="max-width: 900px;" preload="none" poster="/assets/images/20190908_Airtest_Introduction_1.png">
<source id="mp4" src="/assets/videos/20190908_Airtest_Introduction_1.mp4" type="video/mp4">
</video>

网易 18年1月份开源 在内部使用3年 到目前已运行了4年
刘欣 网易游戏QA技术总监 演讲
Google推荐


官方文档：
http://airtest.netease.com/docs/docs_AirtestIDE-zh_CN/index.html

演讲视频：
https://www.youtube.com/watch?v=lJGBDFgOulY

开源地址：
https://github.com/AirtestProject/Airtest

(参考)开源访谈：
https://www.oschina.net/question/3820517_2278684?nocache=1524704966955

![example](/assets/images/20190908_Airtest_Introduction_2.gif)


![structure](/assets/images/20190908_Airtest_Introduction_3.jpg)

UI自动化测试
可以模拟最终用户的动作，测试整个系统。

跨平台

poc

难点是图片识别，两种思路：
* 从图片识别
	两种方法：
		图片像素识别，一定的模糊度（像素出现范围半径）
		OpenCV特征识别
* 从控件树识别
	Windows Poco codedUI
	Selenium

持续集成 UT IT 冒烟测试 Case覆盖率

图片识别  模糊匹配 分辨率

OpenCV 特征识别 字体变化特征

颜色通道 灰度图 

参数
门槛
严格门槛
自定义分辨率变化函数

图片识别工具的使用

控件树 POCO 跨平台支持

