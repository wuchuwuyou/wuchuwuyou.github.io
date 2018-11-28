---
layout: post
title: "macOS Finder Extension"
date: 2018-08-28 17:16:00 +0800
comments: true
categories: technology
keywords: 
published: true
---


# Finder Sync

macOS 上的Finder App 的应用拓展 可以控制显示文件/文件夹的的图标，控制选中文件/文件夹的的右键菜单操作。
在工程中新建target的 选 Finder Sync 就行了，默认会创建一些示例代码。
Finder Sync 和 主app 通信可以使用 URL Scheme 规则进行数据传递，共享数据的话设置一下 App Groups 。也可以主app启动个本地 [http server](https://github.com/robbiehanson/CocoaHTTPServer) 进行通信也可以。

- https://github.com/BokkkRottt/FinderSyncDemo
- https://www.jianshu.com/p/9124340f0f71
- https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Finder.html#//apple_ref/doc/uid/TP40014214-CH15-SW1
- http://www.cocoachina.com/ios/20141004/9824.html
- https://developer.apple.com/design/human-interface-guidelines/macos/extensions/finder-sync-extensions/
 
