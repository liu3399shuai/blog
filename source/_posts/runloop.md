---
title: "iOS的RunLoop简介"
date: 2018-05-22 15:20:25 +0800
comments: true
tags:
- iOS
- RunLoop
categories:
- [iOS]
- [RunLoop]
---

简单介绍下iOS的RunLoop

<!-- more -->

### who
runloop 是线程的基础设施，是线程的一部分，有一定的监听的功能

### what
Each NSThread object, including the application’s main thread, has an NSRunLoop object automatically created for it as needed.

### why
An NSRunLoopobject processes input sources such as mouse and keyboard events from the window system,NSTimer events, NSPort objects, and NSConnection objects. 

### how
do{} while()  

mode : `NSDefaultRunLoopMode`、`NSRunLoopCommonModes`、`NSEventTrackingRunLoopMode`

例子 : 定时器add到defaultMode上，scroll会导致mode切到trackingMode，这样定时器就暂停了
NSURLConnection 下载的时候，要开启runLoop

`CFRunLoop` 是线程安全的，`NSRunLoop` 是线程不安全的

![](/images/runloop.png)

