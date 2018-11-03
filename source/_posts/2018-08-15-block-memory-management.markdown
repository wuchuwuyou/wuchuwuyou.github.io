---
layout: post
title: "block memory management"
date: 2018-08-15 11:52:02 +0800
comments: true
categories: technology
keywords: 
published: true
---
block内存管理，先看几个例子

 

```objective-c
1.
    int a = 0;
    void (^block)(void) = ^ {
        printf("in block a = %d,%p\n",a,&a);
    };
    a = 1;
    block();
    printf("after block a = %d,%p\n",a,&a);

```


```objective-c
2.
    __block int a = 0;
    void (^block)(void) = ^ {
        printf("in block a = %d,%p\n",a,&a);
    };
    a = 1;
    block();
    printf("after block a = %d,%p\n",a,&a);

```


```objective-c
3.
    __block int a = 0;
    void (^block)(void) = ^ {
        printf("in block a = %d,%p\n",a,&a);
        a = 2;
    };
    a = 1;
    block();
    printf("after block a = %d,%p\n",a,&a);
```

输出分别是

```
1.
in block a = 0,0x600000252ec0
after block a = 1,0x7ffee71c8104
2.
in block a = 1,0x600000429ef8
after block a = 1,0x600000429ef8
3.
in block a = 1,0x60000042b118
after block a = 2,0x60000042b118
```

输出中不难看出在1中变量 a 在 block 中被 copy 了生成了新的地址。经过 __block修饰符声明的变量在 block 中就是同一个地址。

