---
layout: post
title: "MAC OS Tips"
date: 2020-10-31 11:00:00 +0800
tags: Mac
---

## 在 dock 中隐藏图标

有事由于某些原因，你不想让 dock 中显示某个 App 的图标。比如不想让人看到你用某个软件，或者像钉钉这样强制显示未读数字的。

- 操作方法：
  1. 在 Finder 中的 Applilcations 找到程序图标，右击程序显示包内容
  2. 找到 Info.plilst，用编辑工具打开
  3. 在`<dict></dict>`之间加入参数`<key>LSUIElement</key><string>1</string>`
  4. 重启 App，dock 中就不再显示了

* 修改后可能影响某些功能
* 如果想恢复显示，只需要删掉增加的参数重启即可
* 如果想正常显示，需要在 Launchpad 中点击。可以把图标快捷方式拉到 dock，以便快速点击显示
