---
layout: post
title: "制作发布自己的pods到CocoaPods"
date: 2015-09-15 17:45:17 +0800
comments: true
categories: technology
---


很早之前制作过自己的pods，以前是然后pull request到specs,最近又想弄了，发现现在不再接受pull request了 改为trunk服务了，所以记录一下，免得自己以后也忘记了。

关于[CocoaPods Trunk](http://blog.cocoapods.org/CocoaPods-Trunk/)这里有介绍。


## 准备
先升级下CocoaPods版本,0.33以上就行了。

	pod --version看一下版本。

然后注册一下trunk

	pod trunk register your email 'your name'
	
```your email```就是填的邮箱了，```your name```就写你的名字。

完了话会收到一份邮件，验证一下。

然后就可以使用
	
		pod trunk me 	
		
来查看一下你自己的注册信息了。

如果你已经制作完了pod并创建修改完了podspec，就直接往下看了，如果没有就在网上搜一下。
## 提交
都弄完了我们就可以push啦。

当然了我们先验证下podspec文件喽

	pod spec lint your.podspec
	
这步要是有问题就看看报错修改喽。没问题就继续
	
	pod trunk push your.podspec
	
成功之后呢，我们看一下有没有

	pod search yourpod
	
就是搜下你提交的pod的名字。
我们也可以在本地```~/.cocoapods```的路径下看一下有没有```yourpod.podspec.json```这个文件。

如果之前已经发布过的pod，作者本人声明所有权可以通过[Claim Your Pods](http://blog.cocoapods.org/Claim-Your-Pods/)来看下具体步骤。[认领地址](https://trunk.cocoapods.org/claims/new)

## End 
制作的时候也会有很多要注意的地方，我实在是懒的很，还是自己搜搜吧。。。。我这个写的相当粗糙了，网上有很多写的相当具体的了。比如：

- [http://nshipster.cn/cocoapods/](http://nshipster.cn/cocoapods/)
- [http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)

-----

### 更新一下:

如果使用
	
	pod trunk XXX
	
报错 
	
	[!] Authentication token is invalid or unverified. Either verify it with the email that was sent or register a new session.
	
那就再执行下命令 

	pod trunk register your email

然后在你的邮箱里找找邮件打开验证一下，再继续。

-----
更新

自己的制作的库要引用其他pod的文件的时候，引用方式头文件报错了，具体就是

	#import<xxxx.h>
	
这类不行了 换成 

	＃import<xxxx/xxxx.h>
	

--EOF--
