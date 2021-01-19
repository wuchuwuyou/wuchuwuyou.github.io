---
title: Qt 显示语言切换
date: 2021-01-19 17:04:22
tags: 
- Qt
categories: technology
keywords:  Qt,translator,国际化,internationalization,i18n
---

核心操作就是通过 ```QTranslator```  加载对应的国际化文件
调用  installTranslator 改变当前显示语言

	#ifndef QT_NO_TRANSLATION
	    	static bool installTranslator(QTranslator * messageFile);
	    	static bool removeTranslator(QTranslator * messageFile);
	#endif

系统有个系统事件

	void MyWidget::changeEvent(QEvent *event)
	{
	    if (event->type() == QEvent::LanguageChange) {
	        titleLabel->setText(tr("Document Title"));
        ...
    	} else
        	QWidget::changeEvent(event);
	}

可以处理显示语言变化的消息

Qt提供的翻译在 Qt安装目录下  ```/translations```  文件夹内 ,直接加到工程资源中就行

### 参考

- [https://wiki.qt.io/How_to_create_a_multi_language_application](https://wiki.qt.io/How_to_create_a_multi_language_application)
- [https://doc.qt.io/qt-5/internationalization.html](https://doc.qt.io/qt-5/internationalization.html)
- [https://doc.qt.io/qt-5/qtranslator.html](https://doc.qt.io/qt-5/qtranslator.html)
- [https://doc.qt.io/qt-5/qlocale.html](https://doc.qt.io/qt-5/qlocale.html)
- [https://stackoverflow.com/questions/36345709/qlocalelanguage-traditional-and-simplified-chinese](https://stackoverflow.com/questions/36345709/qlocalelanguage-traditional-and-simplified-chinese)