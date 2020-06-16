---
title: Qt base64
date: 2020-06-16 23:59:58
tags:
---

```
QString base64Encode(const QString &src)
{
    QByteArray s = src.toUtf8();
    QByteArray ba = s.toBase64();
    return QString(ba);
}


QString base64Decode(const QString &src)
{
    QByteArray s = src.toUtf8();
    QByteArray ba = QByteArray::fromBase64(s);
    return QString(ba);
}
```