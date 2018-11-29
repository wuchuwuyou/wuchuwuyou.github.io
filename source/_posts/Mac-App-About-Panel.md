---
title: Mac App 调用显示关于页面
date: 2018-11-29 22:54:12
tags:
categories: technology
published: true
keywords: About Panel, 关于页面
---

创建新项目的时候系统会自动创建一个显示页面，比如 vs code 的，这都是默认显示的窗口
![VSCode](/images/vscode.png)
但是我们想手动调用这个默认窗口的话，系统也提供了一些方法，可以直接显示出来

```objc
- (void)orderFrontStandardAboutPanel:(id)sender;

```
也可以使用下面这个方法设置窗口里面要显示的内容

```objc
- (void)orderFrontStandardAboutPanelWithOptions:(NSDictionary<NSAboutPanelOptionKey, id> *)optionsDictionary;
```
接受参数是一个字典 key 是 `NSAboutPanelOptionKey`

> [NSAboutPanelOptionApplicationIcon](https://developer.apple.com/documentation/appkit/nsapplication/1428479-orderfrontstandardaboutpanelwith?language=objc)
> [NSAboutPanelOptionApplicationName](https://developer.apple.com/documentation/appkit/nsaboutpaneloptionapplicationname?language=objc)
> [NSAboutPanelOptionApplicationVersion](https://developer.apple.com/documentation/appkit/nsaboutpaneloptionapplicationversion?language=objc)
> [NSAboutPanelOptionCredits](https://developer.apple.com/documentation/appkit/nsaboutpaneloptioncredits?language=objc)
> [NSAboutPanelOptionVersion](https://developer.apple.com/documentation/appkit/nsaboutpaneloptionversion?language=objc)

# 参考
- https://developer.apple.com/documentation/appkit/nsapplication/1428724-orderfrontstandardaboutpanel?language=objc
