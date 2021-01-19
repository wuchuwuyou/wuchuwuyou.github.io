---
title: Qt系统托盘
date: 2020-06-17 00:09:11
categories: technology
tags:
- Qt
---

QSystemTrayIcon

## macOS 上只显示托盘图标
如果只想使用托盘，不想在dock上显示应用的话就在项目中 info.plist 里加上

```
<key>LSUIElement</key>
<string>1</string>
```
