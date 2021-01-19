---
title: Qt 项目集成 Mac Extension
date: 2020-06-17 00:05:10
categories: technology
tags:
- Qt
- macOS
---


1. 直接导出原生Xcode工程的 extension
2. 在 pro 文件中加入关联

```
macx: {
plugins.path = Contents/PlugIns
plugins.files = $$_PRO_FILE_PWD_/../ext/xxx.appex
QMAKE_BUNDLE_DATA += plugins
}
```

> plugins.path 文件输出的应用包内路径

> plugins.files  extension 的本地路径
  

# Finder Sync Communicate 
- https://stackoverflow.com/questions/41016558/how-should-finder-sync-extension-and-main-app-communicate
- https://gist.github.com/hkalexling/1358f9bb2e0708b7bc1ba03daaeb97d0
