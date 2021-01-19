---
title: Qt Windows 窗口去掉右上角问号
date: 2021-01-19 17:23:43
categories: technology
tags:
- Qt
---

在 Widget 中 加上 
```
this->setWindowFlags(windowFlags()&~Qt::WindowContextHelpButtonHint);
```
