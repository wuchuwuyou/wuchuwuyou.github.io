---
title: Codesing & Notarize app on macOS
date: 2021-01-19 16:35:00
tags: macOS,codesign, notarization,公证,签名
---

## Codesign

可用签名证书查询

	security find-identity -v -p codesigning

签名

	codesign -s 'Name of the certificate’ /path/Example.app

重新签名

	codesign -f -s 'Name of the certificate' /path/Example.app
	
应用签名信息

	codesign -vv -d Example.app

应用签名认证

	codesign --verify Example.app

没有任何输出代表签名是完好的

应用可用验证

	spctl -a -v Example.app

## Notarization

- Apple 新认证方式
- Bundle ID 应用bundle ID
- Username Apple ID
- Password Apple ID 密码 是一个一次性的密码
- FilePath 要认证的包 一般是dmg 或者 zip .app 的不行
- Team ID 开发者账号所在的 Team ID 登录到开发者账号 资料里面查看

校验

	xcrun altool --notarize-app --primary-bundle-id “Bundle ID" --username “Username" --password “Password" --file FilePath -itc_provider "Team ID"

上面的执行完成后会生成 ```RequestUUID ```
	

使用下面的方法查看即可

查询 邮件中 ID

	xcrun altool --notarization-info RequestUUID -u “Username --password “Password"


查到认证通过后
需要重新签名一下安装包

重新签一下

	xcrun stapler staple “FilePath"



`stapler validate` will do this -

```
$ stapler validate myfile.pkg 
Processing: myfile.pkg
The validate action worked!

```

- If `The validate action worked!` is printed, the specified pkg file is notarized.
- If `does not have a ticket stapled to it.` is printed, the specified pkg file is either not notarized, or the notarization was never followed up with the [stapling step](https://developer.apple.com/documentation/xcode/notarizing_macos_software_before_distribution/customizing_the_notarization_workflow#3087720).


## 检查公证结果脚本

NotarizationScript.sh

```
# 公证的 bundle id
BUNDLE_ID="xxx"
# apple id
USERNAME="xxx"
# 这里不是apple账号的密码，而是生成的app密匙
PASSWORD="xxxx"
#team id
PROVIDER="xxxx"

filePath=$1

# 公证func 传入文件路径为参数
function uploadFileAndNotarized()
{

	echo "start notarized $filePath ..."

	#上传安装包到苹果服务器进行认证，输出结果到upload_log.txt
	xcrun altool --notarize-app --primary-bundle-id "$BUNDLE_ID" --username "$USERNAME" --password "$PASSWORD" --file $filePath -itc_provider "$PROVIDER" &> upload_result
	# 从日志文件中读取UUID,并隔一段时间检查一次公证结果
	# 只有成功的格式是 RequestUUID = 
	uuid=`cat upload_result | grep -Eo 'RequestUUID = [[:alnum:]]{8}-([[:alnum:]]{4}-){3}[[:alnum:]]{12}' | grep -Eo '[[:alnum:]]{8}-([[:alnum:]]{4}-){3}[[:alnum:]]{12}' | sed -n "1p"`
	# 如果上传过了，则会返回 The upload ID is 
	if [[ "$uuid" == "" ]];then
		uuid=`cat upload_result | grep -Eo 'The upload ID is [[:alnum:]]{8}-([[:alnum:]]{4}-){3}[[:alnum:]]{12}' | grep -Eo '[[:alnum:]]{8}-([[:alnum:]]{4}-){3}[[:alnum:]]{12}' | sed -n "1p"`
		echo "The software asset has already been uploaded. The upload ID is $uuid"
	fi
	echo "notarization UUID is $uuid"
	# 即没有上传成功，也没有上传过，则退出
	if [[ "$uuid" == "" ]]; then
		echo "No success no uploaded, unknown error"
		cat upload_result  | awk 'END {print}'
		return 1
	fi

	while true; do
	    echo "checking for notarization..."
	 
	 	#通过RequestUUID查询服务器工作状态，输出结果到check_status.txt
	    xcrun altool --notarization-info "$uuid" --username "$USERNAME" --password "$PASSWORD" &> check_status
	    checkResult=`cat check_status`
	    
	    t=`echo "$checkResult" | grep "success"`
	    f=`echo "$checkResult" | grep "invalid"`
	    if [[ "$t" != "" ]]; then
	        echo "notarization done!"
	        echo $checkResult

	        echo "do staple $filePath"


	        xcrun stapler staple $filePath &>  stapler_result

	        echo `cat stapler_result`

	        echo "stapler done!"
	        
	        break
	    fi

	    if [[ "$f" != "" ]]; then
	        echo "Failed : $checkResult"
	        return 1
	    fi
	    echo "not finish yet, sleep 1min then check again..."
	    sleep 60
	done
	return 0
}


uploadFileAndNotarized $filePath
```
保存文件 直接调用传入本地包路径即可 

```sh NotarizationScript.sh  filepath```



# 参考

- [https://www.logcg.com/archives/3222.html](https://www.logcg.com/archives/3222.html)
- [https://antscript.com/post/2019-10-08-notarizing-mac-app/](https://antscript.com/post/2019-10-08-notarizing-mac-app/)
- [https://blog.csdn.net/lovechris00/article/details/102309757](https://blog.csdn.net/lovechris00/article/details/102309757)
- [https://developer.apple.com/documentation/xcode/notarizing_your_app_before_distribution?language=objc](https://developer.apple.com/documentation/xcode/notarizing_your_app_before_distribution?language=objc)
- [https://help.apple.com/xcode/mac/current/#/dev88332a81e](https://help.apple.com/xcode/mac/current/#/dev88332a81e)
- [https://help.apple.com/xcode/mac/current/#/dev033e997ca](https://help.apple.com/xcode/mac/current/#/dev033e997ca)
- [https://developer.apple.com/developer-id/](https://developer.apple.com/developer-id/)
- [https://blog.csdn.net/shengpeng3344/article/details/103369804](https://blog.csdn.net/shengpeng3344/article/details/103369804)
- [https://scriptingosx.com/2019/09/notarize-a-command-line-tool/](https://scriptingosx.com/2019/09/notarize-a-command-line-tool/)
- [https://github.com/CaicaiNo/Apple-Mac-Notarized-script/blob/master/Apple-Mac-Notarized-script.sh](https://github.com/CaicaiNo/Apple-Mac-Notarized-script/blob/master/Apple-Mac-Notarized-script.sh)
- [https://blog.csdn.net/ftpleopard/article/details/102721138](https://blog.csdn.net/ftpleopard/article/details/102721138)
