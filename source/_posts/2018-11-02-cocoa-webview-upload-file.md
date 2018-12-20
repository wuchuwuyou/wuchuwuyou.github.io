---
layout: post
title: "Cocoa中WebView允许选择本地上传文件"
date: 2018-11-02 11:30:44 +0800
comments: true
categories: 
- technology
tags: 
- WebView

keywords: Permission denied
---

### Cocoa中WebView允许选择本地上传文件

Cocoa中使用 webview 加载URL显示网页，在进行选择文件上传时候

```js
<input type="file" name="uploadfile">

```

在webview上不能正常的弹出选择器，cef正常。

### WebUIDelegate
解决该问题只需要实现 `WebUIDelegate` 的代理方法 

```objc
- (void)webView:(WebView *)sender runOpenPanelForFileButtonWithResultListener:(id<WebOpenPanelResultListener>)resultListener;

```

### 实现内容

```objc
#pragma mark -  webview
/// <input type="file">
- (void)webView:(WebView *)sender runOpenPanelForFileButtonWithResultListener:(id<WebOpenPanelResultListener>)resultListener {
    // Create the File Open Dialog class.
    NSOpenPanel* openDlg = [NSOpenPanel openPanel];
    
    // Enable the selection of files in the dialog.
    [openDlg setCanChooseFiles:YES];
    
    // Enable the selection of directories in the dialog.
    [openDlg setCanChooseDirectories:NO];
    
    if ([openDlg runModal] == NSOKButton)
    {
        NSArray* URLs = [openDlg URLs];
        NSMutableArray *files = [[NSMutableArray alloc]init];
        for (int i = 0; i <[URLs count]; i++) {
            NSString *filename = [[URLs objectAtIndex:i]relativePath];
            [files addObject:filename];
        }
        
        for(int i = 0; i < [files count]; i++)
        {
            NSString* fileName = [files objectAtIndex:i];
            [resultListener chooseFilename:fileName];
        }
    }
}
```
### 参考
- https://developer.apple.com/documentation/webkit/wkuidelegate/1641952-webview?language=objc
- https://stackoverflow.com/questions/33642521/simple-swift-cocoa-app-with-webkit-upload-picture-doesnt-work/33694264#33694264
- https://stackoverflow.com/questions/7626463/how-to-allow-files-uploading-with-webview-in-cocoa
