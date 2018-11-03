---
layout: post
title: "github Permission denied (publickey)"
date: 2018-11-01 17:19:44 +0800
comments: true
categories: technology
keywords: Permission denied
---

今天重新生成了github的ssh key,添加完以后发现提示 

``` ssh
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
然后查了一下资料，因为我在创建的时候给文件输入了个名字

``` ssh
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/murphy/.ssh/id_rsa): github
```
可以通过```ssh-add ~/.ssh/your_file_name```的方式解决。
也可以在 ~/.ssh/ 目录下编辑config配置文件
```
vi config

Host github.com
 HostName github.com
 User git
 IdentityFile ~/.ssh/github_rsa
```
User 就是用户名

最后验证一下 ```ssh -T git@github.com```
