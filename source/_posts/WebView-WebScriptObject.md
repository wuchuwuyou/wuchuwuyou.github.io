---
title: 一个WebView的循环引用问题
date: 2018-12-17 22:53:39
tags: macOS
categories: technology
published: true
keywords: WebView, WebScriptObject,js,native
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

# 方法废弃
问题是解决了，但是上面的web view 的`- (void)webView:(WebView *)webView windowScriptObjectAvailable:(WebScriptObject *)windowScriptObject`已经废弃了 官方推荐使用 ``- (void)webView:(WebView *)webView didClearWindowObject:(WebScriptObject *)windowObject forFrame:(WebFrame *)frame``;
又但是 WebView 官方也不推荐使用了支持到10.14了，让大家使用 `WKWebView (WK_EXTERN API_AVAILABLE(macosx(10.10), ios(8.0)))`。另推荐使用 `JavaScriptCore`

