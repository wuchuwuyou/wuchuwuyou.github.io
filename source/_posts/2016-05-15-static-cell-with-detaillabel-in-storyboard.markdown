---
layout: post
title: "在 storyboard 中使用静态 Cell 时 Label 内容的显示问题"
date: 2016-05-15 13:04:17 +0800
comments: true
categories: technology
description: "静态cell中Label上内容显示问题"
keywords: static cell,storyboard,detail label
---

这两天遇到一个问题，通过 storyboard 创建的 static cell，通过代码设置 cell 上 label 文字的时候，发现文字经常不能正常显示，只有通过滑动 tableview 之后，重新刷新了以后才能正常显示。
解决办法：
> If you are setting the text to nil somewhere when you try to set it to a non-nil value the actual view that contains the text will be missing. This was introduced in iOS8. Try setting to an empty space @" " character instead.

> This was my issue but it happens when you set it to "" and nil. So you actually have to set it to " ". So basically isEmpty causes this behavior as well.

就是在 storyboard 创建的 cell 中给 label 设置个初始值:“ ”，空格就行了，解决了我遇到的问题。

--EOF--