---
title: 监听键盘鼠标事件
date: 2018-11-22 18:30:45
tags:
published:true
---
# 监听键盘鼠标事件
今儿要做一个监听全局的键鼠事件的操作，之前使用的都是指监听应用内某一个控件的事件
只需要空间是继承 `NSResponder` 的就可以了，按需实现下面的方法就行了
![NSResponder](/images/NSResponder.png)

每个键盘上的按键都有一个对应的编码,下面是针对`command c`和`command v`的处理,`c`和`v`分别对应编码是`8`和`9`,`command`对应的是`NSEventModifierFlagCommand` 。记得把事件传递下去

```
typedef NS_ENUM(unsigned short, LBKeyboardCode) {
LBKeyboardCodeLetterC = 8,
LBKeyboardCodeLetterV = 9,
};
- (void)keyDown:(NSEvent *)event {
if (event.modifierFlags & NSEventModifierFlagCommand) {
NSLog(@"key down event with command:%@",event);
if (event.keyCode == LBKeyboardCodeLetterC) {
// command c
}else if (event.keyCode == LBKeyboardCodeLetterV) {
// command v
}
[self.nextResponder keyDown:event];
}else {
[self.nextResponder keyDown:event];
NSLog(@"key down event:%@",event);
}
}
```

## 创建监听

``` objective-c
CGEventTapLocation tapLocation = kCGHIDEventTap;
CGEventTapPlacement tapPlacement =kCGHeadInsertEventTap;
CGEventTapOptions tapOptions = kCGEventTapOptionDefault;
CGEventMask eventMask = _monitorEventMask();
//        监听某一个进程的，这个就是监听自己的应用的
//    pid_t pid = [[NSProcessInfo processInfo] processIdentifier];
//    self.eventTap = CGEventTapCreateForPid(pid, tapPlacement, tapOptions, eventMask, _captureKeyStroke, (__bridge void * _Nullable)(self));
/// 监听全系统的
self.eventTap = CGEventTapCreate(tapLocation,tapPlacement,tapOptions,eventMask,_captureKeyStroke,(__bridge void * _Nullable)(self));
if (!self.eventTap) {
fprintf(stderr, "failed to create event tap\n");
exit(1);
}


self.runLoopSource =   CFMachPortCreateRunLoopSource(kCFAllocatorDefault, self.eventTap, 0);

CFRunLoopAddSource(CFRunLoopGetCurrent(), self.runLoopSource, kCFRunLoopCommonModes);
CGEventTapEnable(self.eventTap, true);

CFRunLoopRun();

```
## 正常处理

```
CGEventRef _captureKeyStroke(CGEventTapProxy proxy, CGEventType type, CGEventRef event, void* userInfo)
{
/// 不处理的事件
if (type == kCGEventNull | type == kCGEventTapDisabledByUserInput) {
return event;
}
LDKMEventMonitor *self = (__bridge LDKMEventMonitor *)userInfo;
/// 超时的重试
if (type == kCGEventTapDisabledByTimeout) {
NSLog(@"Event Taps Disabled! Re-enabling");
CGEventTapEnable(self.eventTap, true);
return event;
}
/// 转化成 NSEvent 给外部处理
NSEvent *ev = [NSEvent eventWithCGEvent:event];

return event;
}

```

## 超时处理

如果回调的处理时间过长，这个事件会被置为失效，重新设置下可以就可以了。
由于监听的事件调用特别频繁，还是要注意下对于事件处理的效率问题。

``` objective-c
if (type == kCGEventTapDisabledByTimeout) {
NSLog(@"Event Taps Disabled! Re-enabling");         CGEventTapEnable(eventTap, true);
return event;
}

```

## 调试
在.plist 文件添加下面的内容
```
<key>NSAppleEventsUsageDescription</key>
<string></string>
```
调试的时候比较麻烦，
第一次运行会提示：
![accessibilty_alert](/images/accessibilty_alert.png)
需要在```system preferences -> security -> privacy -> accessibility```里面添加并勾选
![accessibility](/images/accessibility.png)
然后每次调试前都要在这个设置里面先删除，然后再添加勾选，特别蛋疼。暂时还不知道有啥好的解决办法

## 参考
- [https://developer.apple.com/documentation/coregraphics/quartz_event_services?language=objc](https://developer.apple.com/documentation/coregraphics/quartz_event_services?language=objc)
- [https://stackoverflow.com/questions/10365487/segmentation-fault-11-cgeventtap-application-stops-processing-mouse-events-aft](https://stackoverflow.com/questions/10365487/segmentation-fault-11-cgeventtap-application-stops-processing-mouse-events-aft)
- [https://forums.developer.apple.com/thread/109283](https://forums.developer.apple.com/thread/109283)
- [https://stackoverflow.com/questions/2969110/cgeventtapcreate-breaks-down-mysteriously-with-key-down-events](https://stackoverflow.com/questions/2969110/cgeventtapcreate-breaks-down-mysteriously-with-key-down-events)
