---
title: Qt 相关资料
date: 2020-04-16 16:47:59
tags:
- Qt
- macOS
categories: technology
---

最近一段时间一直在mac上做客户端开发，自己整理了一些初学Qt的资料。

Qt 安装

- http://download.qt.io/


部分中文文档

- http://www.kuqin.com/qtdocument/index.html
- https://qtguide.ustclug.org/

C++ lambda

- https://www.cnblogs.com/pzhfei/archive/2013/01/14/lambda_expression.html
- https://www.jianshu.com/p/923d11151027

源码

- https://git.merproject.org/mer-core/qtbase


xcode 编译QT

- https://www.jianshu.com/p/8db4e75b8041

Qt 线程和信号槽的应用解释
- https://blog.csdn.net/lutx/article/details/7353957
- https://www.devbean.net/2013/12/qt-study-road-2-thread-and-qobject/

音视频下载测试
- http://samples.mplayerhq.hu/

HTTP Status Code
- https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml


Qt 设置版本号 

You can set the VERSION qmake variable in your pro file:
VERSION = 1.0.0.0

应用中想调用版本号可以这么设置
> DEFINES += APP_VERSION=\\\"$$VERSION\\\"

代码中就可以使用 APP_VERSION 获取版本号了


> 报错 a missing vtable usually means the first non-inline virtual member function has no definition

```
添加 Q_OBJECT 时 记得添加  头文件 #include <QObject>
析构函数
clean 一下，重新构建下
```

> 报错： error: 'staticMetaObject' is not a member of 'YOUR_CLASS_NAME'

```
多继承的话， public QObject 放在写继承那个位置的第一个
```

# Qt系统托盘 
QSystemTrayIcon
# macOS 上只显示托盘图标
如果只想使用托盘，不想在dock上显示应用的话就在项目中 info.plist 里加上

```
<key>LSUIElement</key>
<string>1</string>
```

输入框 QLineEdit 不可编辑

```
    （1）调用lineEdit->setEnabled(false)。确实不可编辑了。不过路径太长时就只能看到后部分了。没关系，再想别的办法就是了。
    （2）setFocusPolicy(Qt::NoFocus); 设置它不可获得焦点。OK.不可编辑，又能查看完整的显示文本。
```
Qt  base64

```
QString base64Encode(const QString &src)
{
    QByteArray s = src.toUtf8();
    QByteArray ba = s.toBase64();
    return QString(ba);
}


QString base64Decode(const QString &src)
{
    QByteArray s = src.toUtf8();
    QByteArray ba = QByteArray::fromBase64(s);
    return QString(ba);
}
```


## Qt关闭最后一个window窗口进程退出问题

```
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    a.setQuitOnLastWindowClosed(false);
}
```

## macOS 打包Qt 工程的问题

- https://doc.qt.io/archives/qt-4.8/deployment-mac.html
- https://blog.csdn.net/robert_cysy/article/details/93979746
- https://www.cnblogs.com/andrewwang/p/8536239.html
- https://blog.inventic.eu/2012/08/how-to-deploy-qt-application-on-macos-part-ii/
- https://codejamming.org/2018/deploy-to-macos
- https://stackoverflow.com/questions/9061354/deploying-qt-frameworks-with-mac-app-and-usage-of-otool
- https://stackoverflow.com/questions/33333628/how-can-i-bundle-a-dylib-within-a-mac-app-bundle-in-qt
- https://stackoverflow.com/questions/20909341/what-is-the-fastest-easiest-way-step-by-step-from-the-beginning-to-code-si/20918932#20918932
- https://blog.csdn.net/liuyez123/article/details/50462637
- https://www.qt.io/blog/2012/04/03/how-to-publish-qt-applications-in-the-mac-app-store-2
- https://doc.qt.io/archives/qt-5.11/osx-deployment.html

## macOS 动态库 问题
- https://www.jianshu.com/p/7f1b50b502d3
- https://www.suninf.net/2015/06/xcode-build-and-paths-config.html
# Qt sqlite dirver not load
- https://blog.csdn.net/guoxiaoqian8028/article/details/29857669
- http://www.voidcn.com/article/p-kmgxdghu-bmn.html

##  如果可以的话，还是使用静态库来的省心 
- https://blog.csdn.net/qq_25188995/article/details/81568491

## Qt 生成 Xcode 工程 
- https://blog.csdn.net/luoyayun361/article/details/79435216


## Qt macOS 应用图标
- https://cheenwe.cn/2016-10-17/qt-deploy/
- https://zhuanlan.zhihu.com/p/75147002
- https://pjw.io/articles/2013/06/19/add-icon-for-qt-app-in-mac-os-x/
- https://www.qt.io/blog/2012/04/03/how-to-publish-qt-applications-in-the-mac-app-store-2
- https://doc.qt.io/qt-5/ios-platform-notes.html
- https://doc.qt.io/archives/qt-4.8/mac-differences.html#translating-the-application-menu-and-native-dialogs

## 签名
签名之前运行

> macdeployqt xxx.app

- http://www.dafscollaborative.org/macuidev.html
- http://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html
- https://successfulsoftware.net/2012/08/30/how-to-sign-your-mac-os-x-app-for-gatekeeper/
- https://nixwang.com/2015/01/07/sign-your-mac-app-with-developer-id/


##  dSYM

- https://stackoverflow.com/questions/9371553/qt-mac-dsym-file-for-a-release-build
- https://www.qt.io/blog/2012/04/03/how-to-publish-qt-applications-in-the-mac-app-store-2
- https://doc.qt.io/archives/qt-4.8/mac-differences.html#translating-the-application-menu-and-native-dialogs

## Url scheme With Qt
- https://blog.csdn.net/weixin_43855938/article/details/88837075
- https://stackoverflow.com/questions/6561661/url-scheme-qt-and-mac
- http://burnttoys.blogspot.com/2008/07/adding-url-scheme-to-qt-application.html
- https://stackoverflow.com/questions/28822086/best-way-to-get-source-url-of-custom-ios-scheme-in-qml

## Qt Notification for macOS
- https://wiki.qt.io/How_to_use_OS_X_Notification_Center
- https://stackoverflow.com/questions/18898725/sending-a-message-to-mountain-lion-notification-centre-with-core-foundation

## Qt macOS 资源文件
- https://wiki.qt.io/Resource_files_in_OS_X_bundle

## Qt macOS 国际化
- https://doc.qt.io/qt-5/macos-issues.html#translating-the-application-menu-and-native-dialogs
- https://forum.qt.io/topic/72977/mac-os-how-to-localize-the-application-menu-entry/5


## Qt 多语言自动化脚本
- https://cloud.tencent.com/developer/article/1463205


## blog 资料

- https://zhuanlan.zhihu.com/llfqt


## qt 拖拽

- https://www.jianshu.com/p/81107b962387
- https://www.devbean.net/2013/05/qt-study-road-2-dnd/
- http://blog.sina.com.cn/s/blog_4a33cfca0100dckg.html


# Qt dylib 相关
- https://forum.qt.io/topic/59209/solved-osx-deployment-fatal-error-with-dylib-library-not-loaded-image-not-found
- https://www.jianshu.com/p/5bf7795db50d

# Qt 上调用 AppDelegate 
- https://www.cnblogs.com/xinghebuluo/p/4857190.html

# Qt 单一应用
- https://stackoverflow.com/questions/5006547/qt-best-practice-for-a-single-instance-app-protection


# Finder Sync Communicate 
- https://stackoverflow.com/questions/41016558/how-should-finder-sync-extension-and-main-app-communicate
- https://gist.github.com/hkalexling/1358f9bb2e0708b7bc1ba03daaeb97d0


# Qt 和 cocoa 混合开发

- https://el-tramo.be/blog/mixing-cocoa-and-qt/

# Qt 项目集成 Mac Extension

1. 直接导出原生Xcode工程的 extension
2. 在 pro 文件中加入关联

```
macx: {
plugins.path = Contents/PlugIns
plugins.files = $$_PRO_FILE_PWD_/../ext/xxx.appex
QMAKE_BUNDLE_DATA += plugins
}
```

> plugins.path 文件输出的应用包内路径

> plugins.files  extension 的本地路径
  