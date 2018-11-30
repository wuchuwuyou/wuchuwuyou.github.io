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

```objc

NSFileManager *fileManager = [NSFileManager defaultManager];
NSMutableDictionary *attributes = [NSMutableDictionary dictionary];
if (isReadOnly) {
//        -r--r--r--
    [attributes setValue:[NSNumber numberWithShort:0444] forKey:NSFilePosixPermissions];
} else {
//        -rwxr--r--
    [attributes setValue:[NSNumber numberWithShort:0744] forKey:NSFilePosixPermissions];
}
NSError *error = nil;
[fileManager setAttributes:attributes ofItemAtPath:path error:&error];

```

Linux一般将文件可存取访问的身份分为3个类别：owner、group、others，且3种身份各有read、write、execute等权限 - 没有权限
-rwxrwxrwx
8进制
r 2*2 4 w 2*1 2 x 2*0 1

-r--r--r-- 0444
-rwxr--r-- 0744

# 参考
- https://linxz.github.io/tianyizone/linux-chmod-permissions.html
