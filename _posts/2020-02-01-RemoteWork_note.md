---
layout: post
title: "远程工作相关总结"
date: 2020-02-01 21:00:00 +0800
tags: Management
---

![remote work](/assets/images/2020-02-01-RemoteWork_note_1.jpg)
近期由于"新冠肺"的快速扩散，很多公司陆续都出台了暂时远程工作的政策。实际上远程工作并非只是逼不得已的选择，应用得当也可以提高整体效率。我曾断断续续有两年的远程工作实践(Web 全栈开发)，在这里总结一下相关经验，也希望能帮助对远程工作有兴趣的人。
这里所说的"远程工作"，并不是指在[Freelancer](https://www.freelancer.com/)或[Upwork](https://www.upwork.com/)这种网站上接活，而只是在传统公司关系中以远程的方式工作。

# 远程团队

有很多公司以远程或半远程的方式上班：

- 37 Signals
  谈到远程工作就不得不说 37 Signals，三位创始人(Jason 等)由于条件所限通过远程方式合作开发网站，后来开发出著名的**Ruby On Rails**框架，逐渐为人们所知。他们 50 名员工分布在世界各地，通过自己摸索的一套管理方法协作，同时他们的产品也是一些为远程协作服务的 WebApp，Basecamp、Campfire 等。

- freelancer
  在线上接活行业比较知名，他们提供的服务是线上撮合项目，抽取中介费、资格考试费等。他们的 50 名员工是以半远程的方式工作，公司允许远程、工作方式也全部线上，但一般每周要到公司一天。

- tower
  在国内提供类似 Basecamp 的功能，在线上进行项目管理。团队成员多数在成都，少数在世界各地，多数坐班，但是工作方式是线上的。(这个 WebApp 我一直在用，功能强大)

- 允许远程的公司有很多，它们的共同特点：
  - 公司本身是互联网公司，提供网上服务，而且往往就是远程工作需要用到的工具
  - 公司规模较小、管理扁平，核心成员往往思路比较新颖，愿意主动推动远程工作
  - 公司招聘很慢，要求严格，缺少主动、自律的员工难以长期远程工作
  - 远程并不是唯一方式，只是允许远程，定期还要面对面开发一段时间，促进沟通

# Why remote? (为什么要远程？)

- 可以一边旅游一边工作，真正"说走就走"
- 可以自由安排工作时间，合理高效的利用时间，只要 4 小时就可以完成原来 8 小时的工作内容
- 不用考虑通勤时间，避免浪费生命在高峰期挤公交/地铁
- 不用周六日去挤商场，可以在工作时间享受各种公共资源
- 节省出更多的空闲时间，可以更多的学习、思考、娱乐、陪家人、锻炼身体
- ...

然而，如果真的只有上面这些好处，大家不都去远程工作了？
在远程时会有很多问题，实际上问题要比上班遇到的多得多：

- 领导对员工的状态难以了解，员工是否在偷懒，无从知道
- 各子项目进度不明，或每个人对进度理解有误，没有机会充分沟通，到最后才发现进度问题
- 团队成员分散在各地，甚至时区都有差异，难以全员随时电话沟通，而办公室随时可以发起会议高效沟通
- 开发过程可能涉及特别昂贵的设备(比如嵌入式开发)，无法满足人手一台远程合作开发
- 由于开发人员可能处于旅行中，工作环境可能不稳定，例如缺少 Wifi、电源、桌椅、安静的环境，而随时发生的应急情况要求有可靠的技术支持
- 工作和生活区域在一起，工作会被家人影响(比如被熊孩子打扰)，工作效率极低
- 一个人在家工作缺少面对面交流，可能存在孤独甚至抑郁
- 没有最基本的运动，例如通勤，长期静止导致身体疾病
- 远程工作并不会缩短工作时间，往往由于没人打断，可能每天都超过 12 小时的工作，时间长了伤害身体
- ...

# 核心问题

人们用远程或通勤的方式聚集在一起，目的是为了 **沟通(Communicate)** 从而共同推进项目完成挑战赚到钱。所以就要考虑沟通的各种要素：

1. 沟通的时机
   是否能够随时发起、需要时结束
   - 公司上班发起较容易，随时"抓人"开会
   - 远程工作这点难以满足，除非提前约定必须在岗的时间并严格遵守
2. 沟通过程的信息量
   对方的语音、语气、表情、姿势、服装、光线、噪音、环境陈设
   - 面对面沟通可以获得所有信息
   - 远程往往带宽不足，只能小窗口，还可能网络不稳定
3. 沟通结果的记录、查看、验证
   脑中的记忆、视频、音频、笔记、邮件、聊天记录，检查项目进度
   - 面对面信息更全面、映像深刻，缺点是可能忘记记录
   - 远程在这方面有优势，核心电子化，随时可以调取

上面这几点，在通勤上班的模式下，有明显的效率优势，这就是为什么主流是上班。
但是随着网络、软件、管理的提高，减小了远程的缺点，远程工作方式的比例在逐步增加。

# 远程工作的思维方式

- 最终创造价值的结果是软件产品，专注的设计、编码、测试才能保证进度和质量。
  - 所以长期来看，各项工作都要降低专注被打断的概率，除非有更好的性价比。例如，制定合理的消息通知机制。
- 沟通是保持方向正确、全员步调一致的基础，而沟通的结果是最重要的，无论什么形式，信息量越大越好。
  - 所以沟通效率当面 > 视频 > 电话 > 即时通信 > 邮件，而沟通的结果要及时记录

# 一些实践方案

# 总结、展望

# 参考
