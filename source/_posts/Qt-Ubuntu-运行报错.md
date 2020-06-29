---
title: Qt Ubuntu 运行报错 /usr/bin/ld: cannot find -lGL
date: 2020-06-29 11:40:45
tags: Qt,ubuntu
---

近期迁移到 ubuntu 上做开发支持，系统版本是 16.04.6

编译项目报错 `/usr/bin/ld: cannot find -lGL` 

讲过查找是缺少 `libGL.so` 库

先本地查找一下 `sudo find / | grep "libGL.so"` 

```
/usr/lib/x86_64-linux-gnu/mesa/libGL.so.1.2.0
/usr/lib/x86_64-linux-gnu/mesa/libGL.so.1
```

找到后 弄个链接到 `usr/lib` 下

`sudo ln -s /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1.2.0 /usr/lib/libGL.so`

如果本地没有查到 `libGL.so` 可以先下载

`sudo apt-get install libgl1-mesa-dev`

然后再弄链接
