---
title: XCTest
date: 2021-01-19 17:37:39
tags:
---

## 异步
处理异步事件 XCTestExpectation

```objective-c
    XCTestExpectation* exception = [self expectationWithDescription:@"metadata"];
    
    NetWrok *req = [NetWrok new] ;
    [req startWithCompletionBlockWithSuccess:^(LDBaseRequest *request) {
        [exception fulfill];
    } failure:^(LDBaseRequest *request) {
        XCTAssertNil(request.error);
        [exception fulfill];
    }];
    
    [self waitForExpectationsWithTimeout:50 handler:^(NSError * _Nullable error) {
        XCTAssertNil(error);
    }];
    
```

## 参考

- [https://nshipster.com/xctestcase/](https://nshipster.com/xctestcase/)
- [https://www.jianshu.com/p/d405a9ca53fe](https://www.jianshu.com/p/d405a9ca53fe)
- [http://blog.kumaya.co/2014/07/10/xctest-assertions/](http://blog.kumaya.co/2014/07/10/xctest-assertions/)