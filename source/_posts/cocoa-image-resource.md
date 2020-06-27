---
title: "iOS工程组件化过程中的资源文件处理"
date: 2019-02-20 15:10:05 +0800
comments: true
tags:
- 组件化
- 资源文件
categories:
- [组件化]
- [资源文件]
---

介绍下ios开发时候工程组件化过程中的资源文件处理

<!-- more -->

###资源文件都有哪些,以下为举例

* .png  
* .xib  
* .nib  
* .bundle 
* .model
* .storyboard

###UIImage api给出的推荐方式

```objc
imageNamed: 一个资源文件的名字

imageNamed:  inBundle: 指定资源文件所在的bundle

imageWithContentsOfFile: 指定资源文件的绝对路径

imageWithData:资源文件的二进制数据
```

###spec.resources 和 spec.resource_bundle 区别

```objc
spec.resources就是把资源文件原封不动的放到工程结构下

spec.resource_bundle 就是把资源文件统一捆绑打包成bundle形式的文件资源包放到工程结构下
```

###资源文件处理方式

1. 使用data形式的资源文件，比如如果是image，可以将资源文件生成二进制，把其二进制data放到代码里面，使用imageWithData方式获取，缺点是可读性差些，因为资源文件使用的时候路径寻址方式，做的不够好

2. 资源文件裸露在工程下，不放在xcasset里面，那么在ipa包里面是裸露放在根目录下，可以直接使用，缺点是没法统一管理

3. 资源文件如果是放在xcasset下，那么在podspec里使用resource命令也没有将资源文件放到根目录的asset.car下，需要使用resource_bundle将其放到bundle下，在代码里指定更改获取资源的路径UIImage imageWithName inBundle 指定bundle资源文件

是否使用use_framework!会对resource的物理路径造成不一样的结果，这个时候解决方法

可以人工更改获取资源文件的bundle的路径，让它同时支持是否使用use_framework!两种状态下的资源路径

也可以将资源文件放回根目录下，比如可以将功能模块拆分为2个子库，一个是代码库，一个是资源库，资源库的podspec里面设置成静态framework ( .static_library = true ) ，这样的话静态framework的话cocoapod会把其资源文件放在ipa包的根目录下，动态framework才会把资源文件嵌到framework里面