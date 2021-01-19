---
title: git remove submodules
date: 2021-01-19 17:30:33
tags: git,submodule
---
path/name 就是本地引入的 submodule 的路径，可以使用```git submodule```  来查看本地路径

依次执行以下命令，来删除本地引用的submodule

	1. git submodule deinit -f -- path/name    
	2. rm -rf .git/modules/path/name
	3. git rm -f path/name
	4. git rm --cached path/name