---
title: macOS 在Main Menu显示已经打开的Window
published: true
categories:
  - technology
date: 2021-06-08 09:37:22
tags: macOS
keywords: Main Menu
---

{% asset_img MainMenu.png Main Menu %}
这个设置是默认的设置

1. NSWindow 属性 ```excludedFromWindowsMenu``` 设置为NO,默认就是NO
2. 设置Window的tittle，没有title不会显示到menu上

#### 参考
- [苹果文档](https://developer.apple.com/documentation/appkit/nswindow/1419175-excludedfromwindowsmenu)
