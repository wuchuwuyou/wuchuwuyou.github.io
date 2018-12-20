---
title: 一个WebView的循环引用问题
date: 2018-12-17 22:53:39
tags: 
- macOS
- WebView
- cocoa
categories: technology
published: true
keywords: WebView, WebScriptObject ,js ,native
---

# macOS WebView JSToNative

项目里通过使用的是WebView 并且通过 `- (void)webView:(WebView *)webView windowScriptObjectAvailable:(WebScriptObject *)windowScriptObject`  这个方法实现JS到Native的通信。

项目中代码如下:

``` objc
self.mainWebView.frameLoadDelegate = self;

# pragma mark -  WebFrameLoadDelegate
- (void)webView:(WebView *)webView windowScriptObjectAvailable:(WebScriptObject *)windowScriptObject {
	[windowScriptObject setValue:self forKey:@"jsToNative"];
}
```

这样会形成循环引用，`self => self.mainWebView => self.mainWebView. windowScriptObject => self`

可以如下修改解决问题：

```objc
self.jsToNativeObject = [[JSToNativeObject alloc] initWithDelegate:self];

# pragma mark -  WebFrameLoadDelegate
- (void)webView:(WebView *)webView windowScriptObjectAvailable:(WebScriptObject *)windowScriptObject {
	[windowScriptObject setValue:self.jsToNativeObject forKey:@"jsToNative"];
}
```
<!--More-->
JSToNativeObject.h


```objc
@interface JSToNativeObject : NSObject

- (instancetype)initWithDelegate:(id<JSToNativeWebViewProtocol>)delegate;

@property (nonatomic,weak,readonly) id<JSToNativeWebViewProtocol>delegate;

@end

```

JSToNativeObject.m


```objc
- (instancetype)initWithDelegate:(id<JSToNativeWebViewProtocol>)delegate {
	self = [super init];
	if (self) {
		self.delegate = delegate;
	}
	return self;
}

- (id)invokeDefaultMethodWithArguments:(NSArray *)arguments {
	if (arguments.count < 2) {
		return nil;
	}
	NSString *methodStr = arguments[0];

	NSString *paramsStr = arguments[1];
	  
	SEL sel;
	if ([methodStr isEqualToString:@"xxxxxx"]) {
		sel = @selector(xxxxxx:);
	}else {
		sel = nil;
	}
	return [self handleSelector:sel withObject: paramsStr];
}

- (id)handleSelector:(SEL)aSelector withObject:(NSDictionary *)params {
	if (aSelector != nil) {
		if ([self.delegate respondsToSelector:aSelector]) {
			// 方法签名(方法的描述)
			NSMethodSignature *signature = [[self.delegate class] instanceMethodSignatureForSelector:aSelector];
			if (signature == nil) {
				NSLog(@"方法签名不存在");
				//可以抛出异常也可以不操作。
			}
# pragma clang diagnostic push
# pragma clang diagnostic ignored "-Warc-performSelector-leaks"
			id obj = [self.delegate performSelector:aSelector withObject:params];
# pragma clang diagnostic pop
			///  有返回值类型，才去获得返回值 void v  objc @
			if (signature.methodReturnLength) {
				return obj;
			}
		}
	}

	return nil;
}

```


### 改进版

上面的代码有崩溃问题，原因见 performSelector 那篇

```objc
- (id)handleSelector:(SEL)aSelector withObject:(NSDictionary *)params {
    if (aSelector != nil) {
        if ([self.delegate respondsToSelector:aSelector]) {
            // 方法签名(方法的描述)
            NSMethodSignature *signature = [[self.delegate class] instanceMethodSignatureForSelector:aSelector];
            if (signature == nil) {
                NSLog(@"方法签名不存在");
                //可以抛出异常也可以不操作。
                return nil;
            }
            if (params == nil) {
                params = @{};
            }
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            
//            id obj = [self.delegate performSelector:aSelector withObject:params];
            ///  有返回值类型，才去获得返回值 void v  objc @
            if (signature.methodReturnLength > 0) {
                return [self.delegate performSelector:aSelector withObject:params];
            }else {
                [self.delegate performSelector:aSelector withObject:params];
            }
#pragma clang diagnostic pop

        }
    }
    
    return nil;
}
```
### 升级版

可以接受并处理多个参数,区分有无返回值

```objc

- (id)handleSelector:(SEL)aSelector withObject:(NSDictionary *)params {
    if (aSelector != nil) {
        if ([self.delegate respondsToSelector:aSelector]) {
            // 方法签名(方法的描述)
            NSMethodSignature *signature = [[self.delegate class] instanceMethodSignatureForSelector:aSelector];
            if (signature == nil) {
                NSLog(@"方法签名不存在");
                return nil;
            }
            if (params == nil) {
                params = @{};
            }
            
            // NSInvocation : 利用一个NSInvocation对象包装一次方法调用（方法调用者、方法名、方法参数、方法返回值）
            NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];
            invocation.target = self.delegate;
            invocation.selector = aSelector;
            
            
            self.objects = @[params];
            // 设置参数
            NSInteger paramsCount = signature.numberOfArguments - 2; // 除self、_cmd以外的参数个数
            paramsCount = MIN(paramsCount, self.objects.count);
            for (NSInteger i = 0; i < paramsCount; i++) {
                id object = self.objects[i];
                if ([object isKindOfClass:[NSNull class]]) continue;
                [invocation setArgument:&object atIndex:i + 2];
            }
            
            // 调用方法
            [invocation invoke];
            
            // 获取返回值
            void *result = NULL;

            if (signature.methodReturnLength) { // 有返回值类型，才去获得返回值
                [invocation getReturnValue:&result];
            }
            
            if (result == NULL) {
                return nil;
            }
            
            id returnValue;
			// 提前释放问题
            returnValue = (__bridge id)result;
            return returnValue;
        }
    }
    
    return nil;
}

```


# 方法废弃
问题是解决了，但是上面的web view 的`- (void)webView:(WebView *)webView windowScriptObjectAvailable:(WebScriptObject *)windowScriptObject`已经废弃了 官方推荐使用 ``- (void)webView:(WebView *)webView didClearWindowObject:(WebScriptObject *)windowObject forFrame:(WebFrame *)frame``;
又但是 WebView 官方也不推荐使用了支持到10.14了，让大家使用 `WKWebView (WK_EXTERN API_AVAILABLE(macosx(10.10), ios(8.0)))`。另推荐使用 `JavaScriptCore`

