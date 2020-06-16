---
title: Qt 设置版本号
date: 2020-06-16 23:53:30
tags:
---

Qt 设置版本号 

You can set the VERSION qmake variable in your pro file:
VERSION = 1.0.0.0

应用中想调用版本号可以这么设置
> DEFINES += APP_VERSION=\\\"$$VERSION\\\"

代码中就可以使用 APP_VERSION 获取版本号了