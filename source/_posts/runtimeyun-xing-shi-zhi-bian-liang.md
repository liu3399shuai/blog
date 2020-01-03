---
layout: post
title: "runtime运行时之变量(variable)"
date: 2016-02-29 18:55:56 +0800
comments: true
tags:
- [variable]
categories:
- [runtime]
- [OC]
---

runtime使用之variable
![](/images/runtime_variable1.png)

<!-- more -->

### runtime使用之variable

![](/images/runtime_variable1.png)

![](/images/runtime_variable2.png)

property

![](/images/runtime_variable3.png)

https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101

每一个  objc_property_t指向的结构体里面 都包含一个 attributions的成员变量，
比如  @property (nonatomic,weak) IBOutlet UITextField
通过 property_getAttributes 获取到，它的attributions是  T@"UITextField",W,N,V_tf
而通过 property_copyAttributeList 获取到以下图片

![](/images/runtime_variable4.png)

![](/images/runtime_variable5.png)
