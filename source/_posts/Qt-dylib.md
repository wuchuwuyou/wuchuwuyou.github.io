---
title: Qt include dylib on macOS
date: 2020-06-17 00:08:14
tags:
---

项目中都是通过 动态库 的方式来组合模块的,debug模式下没有啥问题，因为IDE给处理了很多东西,但是在release模式下，打包出来的应用就有问题了。其中涉及到可执行程序引用dylib ,dylib 之间的相互引用

参看依赖

```
// path/file 你自己要查看文件的路径
otool -L  /path/file
```

一种是修改动态库之间的引用
b.dylib 里面引用到 a.dylib

```
install_name_tool -change a.dylib @executable_path/path/a.dylib b.dylib
```

可执行程序引用动态库

```
install_name_tool -change "a.dylib" "@rpath/a.dylib" /path/exec
```

其中 `@executable_path ` 和 `@rpath` 还有 `@loader_path` 网上有不少说明，具体的就不再这里介绍了，有兴趣的可以自己去搜索看看。

> 如果不知道要那个库的连接，直接双击运行程序，看看提示的`image not found`信息是嘛，也就大概知道了。

> 关于Qt在Mac上的打包相关的内容，博客上也有，参考 `Qt-macOS`相关的内容，由于涉及到的情况还算复杂一点，基本上我遇到的问题都得到了很好的解决，有需要的可以查看一下。


## 参考
- https://forum.qt.io/topic/59209/solved-osx-deployment-fatal-error-with-dylib-library-not-loaded-image-not-found
- https://www.jianshu.com/p/5bf7795db50d
