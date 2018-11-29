---
title: Cocoa中设置文件的权限
date: 2018-11-29 23:56:28
tags:
categories: technology
published: true
keywords: cocoa, 权限
---

在做文件系统的时候需要设置文件权限，之前都是用 `NSTask` 执行脚本设置文件权限，后来发现一个可以使用 ``NSFileManager`` 进行设置。
[NSFilePosixPermissions](https://developer.apple.com/documentation/foundation/nsfileposixpermissions?language=occ)

```
NSNumber *permissions = [NSNumber numberWithUnsignedLong: 493];
NSDictionary *attributes = [NSDictionary dictionaryWithObject:permissions forKey:NSFilePosixPermissions];
NSError *error = nil;
[[NSFileManager defaultManager] setAttributes:attributes ofItemAtPath:path error:&error];

```
