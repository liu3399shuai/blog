---
title: "ios 自动化打包部署"
date: 2018-12-1 19:00:07 +0800
comments: true
tags:
- 自动化打包部署
categories:
- [自动化打包部署]
---

介绍下iOS 自动化打包部署

<!-- more -->

###前沿

app开发完成提测后，受到打包给测试太麻烦，占用开发者时间和电脑，现在都是使用Jenkins平台完成iOS 自动化打包，并自动部署到 分发平台，如蒲公英，appstore等平台

###自动化打包部署shell脚本内容

1. 拉取代码

```shell
// 强制使用远程代码覆盖本地，保证本地代码和服务器的一致性
git fetch —-all 
git reset —-hard

// 拉取代码
git pull
```

2. 打包生成ipa包

```shell
// xcodebuild 是 Xcode的命令行工具

xcodebuild clean

xcodebuild -workspace *.xcwork*  -scheme 'app' -destination generic/platform=iOS -archivePath /Users/Desktop/app/project/archive/app.xcarchive archive

xcodebuild -exportArchive -exportOptionsPlist /Users/Desktop/app/cmd/exportOptions.plist -archivePath /Users/Desktop/app/project/archive/app.xcarchive -exportPath /Users/Desktop/ipa/app${DATE}  -allowProvisioningUpdates

其中 exportOptions.plist 里面指明了 签名方式 (ad-hoc 、app-store、devemlopment)

```

3. 上传至 fir 、appstore 等平台

使用xcrun altool上传AppStore方式 (altool 是 application loader 的命令行工具)

```shell
alias altool='/Applications/Xcode.app/Contents/Applications/Application\ Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool'
```

验证

```shell
altool --validate-app -f /Users/Desktop/app/ipa/app${DATE}/app.ipa -u email -p password --output-format xml
```

上传

```shell
altool --upload-app -f /Users/Desktop/app/ipa/app${DATE}/app.ipa -u email -p password --output-format xml

xcrun altool --validate-app -f xxx/xxx/xxx.ipa -t ios --apiKey xxxxxxxx --apiIssuer xxxxxx --verbose --output-format xml

xcrun altool --upload-app -f xxx/xxx/xxx.ipa -t ios --apiKey xxxxxxxx --apiIssuer xxxxxx --verbose  --output-format xml 
```

使用fir上传fir.im网站 (fir 命令 是 fir.im网站开发的，需要去官网下载支持)

```shell
fir login token

fir p /Users/Desktop/app/ipa/app${DATE}/app.ipa
```
