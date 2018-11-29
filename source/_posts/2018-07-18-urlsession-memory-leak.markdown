---
layout: post
title: "一个内存泄露问题"
date: 2018-07-18 12:38:42 +0800
comments: true
categories: technology
keywords: URLSession,Memory,Leaks,内存泄露
---

### 问题的出现
发现在文件传输的过程中，内存一直在增长并且不释放，反复的测试发现是在文件上传过程中，上传文件块过程中内存不释放，导致内存泄漏。
<!--more-->
### 定位问题
通过 Instruments Leaks 观察定位到是一块儿申请的空间没有释放，这个工具最后显示出来的调用栈显示申请的内存没有释放，导致内存泄露。

```objc
 NSData *data = [LBFileManager readDataOffset:self.blockInfo.location withLength:self.blockInfo.length from:path error:error];
```

指向的是这个函数内部
![内存申请的地方](/images/1529996692544.jpg)



最开始一直以为是这块儿的问题，修改了函数入参
![改写的方法](/images/1529997183341.jpg)

想的是外部传入赋值的对象，交给外部控制（实际上其实这块儿没有影响）。
经过反复测试，发现并不是这块儿的问题。测试的过程中发现存储该对象的地方已经移除了持有了该对象，理论上外部不再持有该对象，但是该对象并没有被释放，没有调用dealloc方法。在和同事的讨论过程中，发现这是个循环引用问题，外部持有者已经不再持有该对象，但是该对象内部形成了循环引用，导致不能释放。

但是我检查了几遍代码并没有发现循环引用的问题。于是使用了return大法，一个个的函数测试，最终定位到读取文件数据发起网络请求后内存泄露。

![网络请求](/images/1530000501732.jpg)


经过一番查找，发现官方文档里有这样一句话

 >The session object keeps a strong reference to the delegate until your app exits or explicitly invalidates the session. If you don’t invalidate the session, your app leaks memory until it exits.

意思就是 session 对象会对代理进行强引用，如果不使 session 失效就会出现内存泄露。
### 解决办法
在调用网络任务结束的地方

![网络请求](/images/session_complete.png)


调用以下方法

```objc
/* -finishTasksAndInvalidate returns immediately and existing tasks will be allowed
 * to run to completion.  New tasks may not be created.  The session
 * will continue to make delegate callbacks until URLSession:didBecomeInvalidWithError:
 * has been issued. 
 *
 * -finishTasksAndInvalidate and -invalidateAndCancel do not
 * have any effect on the shared session singleton.
 *
 * When invalidating a background session, it is not safe to create another background
 * session with the same identifier until URLSession:didBecomeInvalidWithError: has
 * been issued.
 */

- (void)finishTasksAndInvalidate; 

/* -invalidateAndCancel acts as -finishTasksAndInvalidate, but issues
 * -cancel to all outstanding tasks for this session.  Note task 
 * cancellation is subject to the state of the task, and some tasks may
 * have already have completed at the time they are sent -cancel. 
 */
- (void)invalidateAndCancel;
```

### 更新
每次创建的session 并不会随着调用```finishTasksAndInvalidate 或者invalidateAndCancel``` 而释放内存，虽然session已经置为nil但是占用的内容缓存并不会释放掉（[要过10多分钟才能释放掉](https://developer.apple.com/library/archive/qa/qa1727/_index.html)，这是官方上说的我，我试了几次等了半小时也没有释放）
查了不少资料，都是要用单例去解决这个问题。

临时解决办法，设置 ```NSURLSessionConfiguration``` 的 ```URLCache```

```objc
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    //设置请求超时为10秒钟
    configuration.timeoutIntervalForRequest = 10;
    //在蜂窝网络情况下是否继续请求（上传或下载）
    configuration.allowsCellularAccess = NO;
    configuration.URLCache = [[NSURLCache alloc] initWithMemoryCapacity:0 diskCapacity:0 diskPath:nil];
    // 构造NSURLSession，网络会话；
    _session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];
```
是解决了内存的问题，但是不使用缓存又很慌，毕竟应用场景是有大量的文件传输

-------
### 参考
- https://stackoverflow.com/questions/32878257/ios-memory-management-issue-with-arc
- https://developer.apple.com/documentation/foundation/url_loading_system
- https://developer.apple.com/documentation/foundation/urlsession
- https://stackoverflow.com/questions/29514759/uploading-uiimage-data-to-the-server-getting-memory-peaks
- http://www.drdobbs.com/architecture-and-design/memory-leaks-in-ios-7/240168600
- https://blog.csdn.net/jsjxiaobing/article/details/41828419
- https://stackoverflow.com/questions/28223345/memory-leak-when-using-nsurlsession-downloadtaskwithurl/35757989#35757989

