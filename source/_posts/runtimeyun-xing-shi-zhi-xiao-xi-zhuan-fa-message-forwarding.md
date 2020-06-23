---
layout: post
title: "runtime运行时之消息转发(message forwarding)"
date: 2016-02-29 18:56:57 +0800
comments: true
tags:
- [消息转发]
categories:
- [runtime]
- [OC]
---

OC是一门动态语言，在调用`[self performSelector:@selector(dynamic1)]`;  编译时候，可以通过，只有在执行时候，系统才会去找dynamic1方法对应的implement，若找不到这个执行，则就进行消息转发机制流程

<!-- more -->

### runtime之消息转发(message forwarding)

OC是一门动态语言，在调用`[self performSelector:@selector(dynamic1)]`;  编译时候，可以通过，只有在执行时候，系统才会去找dynamic1方法对应的implement，若找不到这个执行，则就进行消息转发机制流程

```objc
[self performSelector:@selector(dynamic1)];  // 这个invoke编译时候将转换为objc_msgSend()
```

![](/images/runtime_forwarding1.png)


![](/images/runtime_forwarding2.png)

* 在一个函数找不到时，Objective-C提供了三种方式去补救：
	1. 调用resolveInstanceMethod给个机会让类添加这个实现这个函数
	2. 调用forwardingTargetForSelector让别的对象去执行这个函数
	3. 调用methodSignatureForSelector（函数符号制造器，）和forwardInvocation（函数执行器）灵活的将目标函数以其他形式执行。

最后，如果都不中，调用`doesNotRecognizeSelector`抛出异常。

参考  <http://www.cnblogs.com/biosli/p/NSObject_inherit_2.html>

![](/images/runtime_forwarding3.png)

在第三步调用 `-(void)forwardInvocation:(NSInvocation *)anInvocation`   之前为什么要调用
这个方法呢 `-(NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector`  ，因为method 由 sel 、type、implement  组成的，只知道一个sel肯定是不行的，所以通过MethodSignature方法给sel一个标签，就是定义下它的参数返回值类型。
