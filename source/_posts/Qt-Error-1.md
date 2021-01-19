---
title: Qt报错 a missing vtable usually means the first non-inline virtual member function has no definition
date: 2020-06-16 23:56:06
categories: technology
tags:
- Qt
---


> 报错 a missing vtable usually means the first non-inline virtual member function has no definition

```
添加 Q_OBJECT 时 记得添加  头文件 #include <QObject>
析构函数
clean 一下，重新构建下
```

> 报错： error: 'staticMetaObject' is not a member of 'YOUR_CLASS_NAME'

```
多继承的话， public QObject 放在写继承那个位置的第一个
```