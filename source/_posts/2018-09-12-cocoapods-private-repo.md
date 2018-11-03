---
layout: post
title: "CocoaPods私有库"
date: 2018-08-28 17:16:00 +0800
comments: true
categories: technology
keywords: 
published: true
---

#CocoaPods私有库
### 私有库版本库
- 创建版本库（repo）私有库的一个索引 建一个git空仓库
- 执行``Pod repo list``列出本地的所有的版本库
- 执行``pod repo add repo的名字 git地址``将索引库添加到本地

### 创建私有库项目
创建项目``pod lib create <项目名称>`` Assets 资源文件 Class 源码文件。
根据命令行提示一步步执行，然后就写代码招呼吧。
提交代码的时候记得添加[tag](./2018-09-12-git-tag.md)一起提交 与 ``.podspec``里面的version一致
``pod lib lint``  检查配置文件，通过以后可以提交
``pod repo push  版本库名称 组件库名字.podspec``
> pod lib lint 和 pod spec lint 命令分别是对podspec的本地校验和远程校验

### 配置依赖
自己写的cocoapods库中引用其他第三方库只需要在``.podspec``文件中设置 ``s.dependency 'AFNetworking', '~> 2.3'``这样的就可以了。但是如果你依赖的是自己的私有库，就会有点问题，需要麻烦一点弄一下。
### 依赖私有库
私有库配置依赖中引用其他私有库，在``.podspec`` 中的写法和上面一样 ``s.dependency 'LDKeychain', '~> 0.1.0' |LDKeychain就是我自己的之前写的私有库``，通过 dependency 引用私有库文件。

但是我们在验证``.podspec``和往自己的私有repo中提交的时候会报错。
只需要如下操作 
> DeviceUniqueKey.podspec 为我的创建的私有库，repo_name是版本库的名字

需要在 pod lib lint 命令是加上 --sources repo地址。例如：
``pod lib lint DeviceUniqueKey.podspec --sources=https://github.com/CocoaPods/Specs.git,xxx.xxx.xxx.xxx/iOS/repo_name.git``

pod repo push 时也需要在命令上加上--sources 例如：
``pod repo push lenovo-specs-repo DeviceUniqueKey.podspec —sources=ssh://git@xxx.xxx.xxx.xxx/iOS/repo_name.git``

完事记得``pod repo update 版本库名称``一下
 