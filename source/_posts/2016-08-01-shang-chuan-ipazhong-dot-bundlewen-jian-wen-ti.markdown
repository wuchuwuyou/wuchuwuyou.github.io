---
layout: post
title: "上传ipa包.Bundle文件问题"
date: 2016-08-01 15:17:46 +0800
comments: true
categories: technology
keywords: ERROR ITMS-90166,ERROR ITMS-90171,Bundle,iOS静态资源包,faceid
---
#### 起因
前些日子集成了某人脸识别的 SDK，通过他们发来的文档成功接入了，发测试也是使用了蒲公英进行分发的，通过 Xcode 打包出 ipa 然后传到蒲公英，一切都是很正常。直到前两天我要提交审核了，上传 ipa 的时候报错了。

报错信息:

>ERROR ITMS-90166: "Missing Code Signing Entitlements. No entitlements found in bundle 'com.xx.xxxx' for executable 'Payload/xx.app/PlugIns/xxxx.bundle/xxxx'.""

>ERROR ITMS-90171: "Invalid Bundle Structure - The binary file 'xx.app/PlugIns/xxxx.bundle/xxxx' is not permitted. Your app can’t contain standalone executables or libraries, other than the CFBundleExecutable of supported bundles. Refer to the Bundle Programming Guide at https://developer.apple.com/go/?id=bundle-structure for information on the iOS app bundle structure."


基本上就是报这两个错误，由于文件都是别人提供的，我就去问了下技术支持，但是没人鸟我啊，所以只能自己解决了。

刚看到报错信息我就想当然的就去网上搜了。费了很长时间都没有解决，焦头烂额，人也很烦躁。

后来实在没招了，仔细看了下报错信息，再对比下源文件，bundle 包下有一个可执行文件，报错信息中也确实给指出了问题:
>Your app can’t contain standalone `executables` or libraries, other than the CFBundleExecutable of supported bundles.

#### 解决办法：

1. 删除编译 bundle 的 info.plist 中 Executable file
2. 删除 bundle 所有的可执行文件。


因为 bundle 是静态的，也就是说，我们包含到包中的资源文件作为一个资源包是不参加项目编译的。也就意味着，bundle包中不能包含可执行的文件。它仅仅是作为资源，被解析成为特定的2进制数据。

还在研究项目中引用的工程是怎么自己把执行文件也打包到 bundle 中的，等研究明白了再做更新。
#### 总结
还是自己太不稳当了，太毛躁，遇到问题就知道网上查，其实问题已经很明显了，稍微用下心就能很快的解决。一定要记得看仔细看报错信息，答案都在里面了。


--EOF--