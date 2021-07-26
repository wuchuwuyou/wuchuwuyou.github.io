---
title: vcpkg 支持 macOS10.10 设置
published: true
categories:
  - technology
date: 2021-07-26 22:15:19
tags: vcpkg
keywords: vcpkg,macOS
---


找到文件 ```/triplets/x64-osx.cmake```
添加下面内容

```
set(VCPKG_OSX_DEPLOYMENT_TARGET 10.10)
set(VCPKG_C_FLAGS -mmacosx-version-min=10.10)
set(VCPKG_CXX_FLAGS -mmacosx-version-min=10.10)
```

这个设置只支持使用 cmake  编译的库。
如果要使用其他库需要目前最好的办法是下载对应的 ports。

