---
title: AppleScript 获取应用路径
date: 2021-01-19 17:02:31
categories: technology
keywords:  AppleScript
---

```
on getAppPath(AppName)
	try
		set launchServicesPath to "/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/LaunchServices.framework/Versions/A/Support/lsregister"
		
		-- get the path to all executables in the Launch Service Registry that contain that appName
		set appPaths to paragraphs of (do shell script launchServicesPath & " -dump | grep --only-matching \"/.*\\" & AppName & "\"")
		log appPaths
		return appPaths
		
	on error
		return "NOT INSTALLED"
	end try
end getAppPath

```


参考:

- [https://www.satimage.fr/software/en/smile/external_codes/file_paths.html](https://www.satimage.fr/software/en/smile/external_codes/file_paths.html)