---
title: "iOS工程组织结构(cocoapods管理私有化仓库)"
date: 2019-02-19 17:50:10 +0800
comments: true
tags:
- 私有化仓库
- cocoapods
- 工程组织结构
categories:
- [私有化仓库]
- [cocoapods]
- [工程组织结构]
---

介绍下ios开发过程中使用cocoapods管理私有化仓库的实现

<!-- more -->

###先看一个现成的工程组织结构事例

![](/images/cocoapods_1.png)

该工程组织形式，是2个xcodeproj ，一个是 app，一个是 pods；
app target下只有 appdelegate的初始化工作，和 image的icon、启动图

pods里面有 Development Pods、Pods；Development Pods里面是开发的业务子仓库，Pods是成熟的第三方工具子仓库、私有工具子仓库

所以，这个工程组织结构，物理上是 一个 主工程仓库，N 个 子仓库，子仓库的管理在podfile里面，如果是 在Development Pods下，则使用:path ,指明podspec，如果是在pods下，则使用 :git，指明 branch、subspecs等

###工程子仓库

* 分类
	1. 外部第三方工具库，开源
	2. 公司内部工具库，私有仓库
	3. 公司内部业务库，私有仓库
	
* 创建
	原则：按照功能来建立，仓库内有耦合，仓库间无耦合
	
	实现：一个可以通过cocoapod来使用的仓库，最简单的就是在一个普通git仓库里加入一个.podspec文件就可以，在podspec里面配置该仓库的spec，name，dependency等等属性，比如参考afnet的podspec <https://github.com/AFNetworking/AFNetworking/blob/master/AFNetworking.podspec>
	以下为测试demo podspec
	
	![](/images/cocoapods_2.png)
	
	仓库的podspec创建好了后，就可以进行验证仓库的pod配置是否正确(终端cd到当前仓库，输入 pod lib lint)，正确结果如下
	
	![](/images/cocoapods_2.png)
	
	验证OK后，就可以将仓库提交到远程服务器了
	
* 使用
	
	podfile通过:path方式，可以给pod生成一个Development Pods文件夹，如下
	
	```objc
	pod 'module_login', :path => "./Module/module_login.podspec”
	```
	
	podfile也可以使用:git方式
	
	```objc
	pod 'module_login', :git => 'git@icode.app.com:app/module_login.git'
	```
	
	
	
	