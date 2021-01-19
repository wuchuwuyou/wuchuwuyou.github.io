---
title: Qt QLineEdit 输入框不可编辑
date: 2020-06-17 00:10:18
categories: technology
tags:
- Qt
---

```
    （1）调用lineEdit->setEnabled(false)。确实不可编辑了。不过路径太长时就只能看到后部分了。没关系，再想别的办法就是了。
    （2）setFocusPolicy(Qt::NoFocus); 设置它不可获得焦点。OK.不可编辑，又能查看完整的显示文本。
```