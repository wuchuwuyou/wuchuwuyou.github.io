---
title: iOS/macOS设置代理
published: true
categories:
  - technology
date: 2021-07-26 22:09:27
tags: macOS
keywords: iOS,macOS,proxy
---

## NSURLSessionConfiguration 方式
URLSession 可以设置 configuration ， NSURLSessionConfiguration 有设置代理的方法，代码如下

```
NSURLSessionConfiguration *conf = [NSURLSessionConfiguration defaultSessionConfiguration];
NSString* proxyHost = @"";
NSNumber* proxyPort = @(0);
NSString* proxyUser = @"";
NSString* proxyPwd  = @"";
NSMutableDictionary *proxyDict = [NSMutableDictionary dictionary];
[proxyDict setObject:[NSNumber numberWithInt:1] forKey:(NSString *)kCFNetworkProxiesHTTPEnable];
[proxyDict setObject:proxyHost forKey:(NSString *)kCFNetworkProxiesHTTPProxy];
[proxyDict setObject:proxyPort forKey:(NSString *)kCFNetworkProxiesHTTPPort];

[proxyDict setObject:[NSNumber numberWithInt:1] forKey:(NSString *)kCFNetworkProxiesHTTPSEnable];
[proxyDict setObject:proxyHost forKey:(NSString *)kCFNetworkProxiesHTTPSProxy];
[proxyDict setObject:proxyPort forKey:(NSString *)kCFNetworkProxiesHTTPSPort];

[proxyDict setObject:proxyUser forKey:(NSString *)kCFProxyUsernameKey];
[proxyDict setObject:proxyPwd forKey:(NSString *)kCFProxyPasswordKey];
conf.connectionProxyDictionary = proxyDict;
  // 需要认证的代理连接 需加上下面的请求头
NSString* proxyIDPasswd = [NSString stringWithFormat:@"%@:%@",proxyUser,proxyPwd];
NSData* proxyoriginData = [proxyIDPasswd dataUsingEncoding:NSUTF8StringEncoding];
NSString *base64EncodedCredential = [proxyoriginData base64EncodedStringWithOptions:0];
NSString *authString = [NSString stringWithFormat:@"Basic: %@", base64EncodedCredential];
conf.HTTPAdditionalHeaders = @{@"Proxy-Authorization": authString};
```

## NSURLProtocol

推荐使用苹果官方的demo [CustomHTTPProtocol](https://developer.apple.com/library/archive/samplecode/CustomHTTPProtocol/Introduction/Intro.html)

需要注意的请求的过滤条件和转发规则，具体原理后面的链接有提供。

## WKWebView 
由于没有用到 WKWebView ，不做过多介绍了，网上有很多教程

## NetworkExtension
需要单独申请权限，并且比较重，不建议使用。不过用起来还是很爽的，多看文档。


## Ubuntu 搭建代理服务器
搭建一个测试环境
主要是用的是一个 squid 服务
需要注意在设置账号密码的时候，在配置文件中一定要确认好认证账号文件的路径

测试 命令 使用curl 测试代理联通性
```
 curl -x http://proxy_server:proxy_port --proxy-user username:password -L http://url
```
具体搭建过程，下方参考中有

## 参考

- [https://www.cnblogs.com/ZhangShengjie/p/12188377.html](https://www.cnblogs.com/ZhangShengjie/p/12188377.html)
- [https://zhuanlan.zhihu.com/p/22043351](https://zhuanlan.zhihu.com/p/22043351)
- [http://www.cocoachina.com/articles/10765](http://www.cocoachina.com/articles/10765)
- [https://nemocdz.github.io/post/ios-设置代理proxy方案总结/](https://nemocdz.github.io/post/ios-%E8%AE%BE%E7%BD%AE%E4%BB%A3%E7%90%86proxy%E6%96%B9%E6%A1%88%E6%80%BB%E7%BB%93/)
- [https://nshipster.cn/nsurlprotocol/](https://nshipster.cn/nsurlprotocol/)
- [https://draveness.me/intercept/](https://draveness.me/intercept/)
- [https://www.cnblogs.com/ssgeek/p/12302135.html](https://www.cnblogs.com/ssgeek/p/12302135.html)
- [http://cn.linux.vbird.org/linux_server/0420squid.php](http://cn.linux.vbird.org/linux_server/0420squid.php)
- [https://www.cnblogs.com/blxt/p/14501176.html](https://www.cnblogs.com/blxt/p/14501176.html)
