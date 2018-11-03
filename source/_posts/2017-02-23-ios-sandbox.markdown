---
layout: post
title: "iOS SandBox"
date: 2017-02-23 11:09:04 +0800
comments: true
categories: technology
keywords: iOS,沙盒,SandBox
---
经常使用 SandBox ,但是总也记不住，记录下来免的又忘记了。
### 沙盒目录
沙盒的目录中有很多文件，经常打交道的是Data文件，它包含下面三个文件

1. Documents：用于存储用户数据，iTunes备份和恢复的时候会包括此目录，所以，苹果建议将程序中建立的或在程序中浏览到的文件数据保存在该目录下。
2. Library：包含两个子目录：Caches 和 Preferences。Caches用来存放用户需要换成的文件。Preferences是APP的偏好设置，可以通过NSUserDefaults来读取和设置。
3. tmp： 用于存放临时文件，这个可以放一些当APP退出后不再需要的文件。

### 获取沙盒路径
#### 根目录
	
	//获取沙盒根目录
	NSString *directory = NSHomeDirectory();
	
#### Documents路径

	//获取Documents路径
	NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
	NSString *path = [paths objectAtIndex:0];
	
#### Library路径
	
	//获取Library路径
	NSArray *paths = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES);
	NSString *path = [paths objectAtIndex:0];
	
#### Caches路径
	
	//获取Caches路径
	NSArray *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
	NSString *path = [paths objectAtIndex:0];
	
#### tmp路径

	//获取tmp路径 
	NSString *tmp = NSTemporaryDirectory();
	
	
--EOF--



