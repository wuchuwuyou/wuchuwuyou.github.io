---
title: Qt关闭最后一个window窗口进程退出问题
date: 2020-06-16 23:58:39
tags:
---

```
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    a.setQuitOnLastWindowClosed(false);
}
```