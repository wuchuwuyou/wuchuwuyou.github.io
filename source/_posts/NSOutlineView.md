---
title: NSOutlineView
date: 2019-01-05 13:48:37
tags: 
- NSOutlineView
- macOS
categories: technology
published: true
keywords: NSOutlineView, reloadData, reloadItem, reloadItem:reloadChildren

---


在使用 `NSOutlineView` 实现文件结构目录的时候，发现在10.12以下版本，首次加载目录时候不能正常的显示出来，滑动一下页面就可以显示了。
当时使用的刷新方法是 `reloadItem:` 和 `reloadItem:reloadChildren:` 

`NSOutlineView` 是继承 `NSTableView` 的，所以就试了下调用了 `reloadData ` 就能正常显示了，查了下文档，文档上就有说明

> Important

> It is possible that your data source methods for populating the outline view may be called before awakeFromNib if the data source is specified in Interface Builder. You should defend against this by having the data source’s outlineView:numberOfChildrenOfItem: method return 0 for the number of items when the data source has not yet been configured. In awakeFromNib, when the data source is initialized you should always call reloadData.

通过xib加载的视图，当数据初始化完成后，你需要调用`reloadData`正常展示数据
