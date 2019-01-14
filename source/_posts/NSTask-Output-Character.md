---
title: NSTask Output Character
date: 2019-01-14 10:23:22
tags:
---
```objc
NSTask *task = [[NSTask alloc] init];
NSMutableDictionary * e = [NSMutableDictionary dictionaryWithDictionary:[[NSProcessInfo processInfo] environment]];
[e setObject:@"en_US.UTF-8" forKey:@"LC_ALL"];
[e setObject:@"en_US.UTF-8" forKey:@"LANG"];

//    [e setObject:@"zh_CN.UTF-8" forKey:@"LC_ALL"];
//    [e setObject:@"zh_CN.UTF-8" forKey:@"LANG"];
//
[task setEnvironment:e];
```

中文字符编码问题 
\xe5\x90\x84\xe7\xa7

添加上面的 environment 就可以了
