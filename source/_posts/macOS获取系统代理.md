---
title: macOS获取系统代理
published: true
categories:
  - technology
date: 2021-06-08 18:13:07
tags:
keywords:
---


```
CFDictionaryRef proxies = SCDynamicStoreCopyProxies(NULL);
if (proxies) {
    NSDictionary *dict = [NSDictionary dictionaryWithDictionary:(__bridge NSDictionary *)proxies];
    CFNumberRef hpEnableRef = CFDictionaryGetValue(proxies, kSCPropNetProxiesHTTPEnable);
    NSNumber *enable = (__bridge NSNumber *)hpEnableRef;
    if (enable) {
        // host
        CFStringRef hpURLRef = (CFStringRef)CFDictionaryGetValue(proxies,kSCPropNetProxiesHTTPProxy);
        // port
        CFNumberRef hpPortRef = CFDictionaryGetValue(proxies, kSCPropNetProxiesHTTPPort);
        
        if (hpURLRef) {
            NSNumber *port = (__bridge NSNumber *)hpPortRef;
            NSString *url = (__bridge NSString *)hpURLRef;
            NSString *pUrl = [NSString stringWithFormat:@"http://%@:%@",url,port];
            self.proxyServer = pUrl;
        }
    }
    CFRelease(proxies);
}

```
