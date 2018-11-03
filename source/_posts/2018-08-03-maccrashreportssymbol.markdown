---
layout: post
title: "macOS崩溃日志符号化"
date: 2018-08-03 09:58:40 +0800
comments: true
categories: technology
keywords: macOS,crash report,symbol
published: true
---

### 分析工具
网上一大堆文章都是分析iOS的崩溃日志的，针对Mac的特别少，其实基本原理都差不多，就不再详细的描述了。就推荐一个崩溃日志符号化的工具。

[下载地址](https://mahdi.jp/apps/macsymbolicator)

[源码地址](https://github.com/inket/MacSymbolicator)

- 使用方法：
	1. 将 .crash 文件和 .dSYM 文件放在同一目录下。
	2. 将 .crash 或者 .dSYM 拖入这个工具中，另外一个文件会自动导入
- 注1：.dSYM 可以在打包时候生产的 .xcarchive 包中找到。
- 注2：注意 .crash 和 .dSYM 的UUID要一致
- 查看dSYM UUID  ``` xcrun dwarfdump --uuid file_path```
- .crash 查看UUID 在 Binary Images 的下一行 包名 后面的那一串字符 

### 还有一个工具可以参考

[dSYMTools](https://github.com/answer-huang/dSYMTools)
这个工具也可以符号化崩溃日志，需要手动操作，具体使用方式链接上面说的很清楚了。

### 参考链接
- http://www.smileykeith.com/2013/08/09/os-x-crash-symbolication/
- https://stackoverflow.com/questions/5842147/how-to-symbolicate-mac-osx-crash-reports-issued-by-apple