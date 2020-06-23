---
layout: post
title: "runtime运行时之方法method"
date: 2016-02-29 18:56:07 +0800
comments: true
tags:
- [method]
categories:
- [runtime]
- [OC]
---

a Method is a combination of a selector and an implement

method 组成结构体

<!-- more -->

### runtime之方法(method)

runtime 参考<https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/>

a Method is a combination of a selector and an implement

method 组成结构体

![](/images/runtime_method1.png)


![](/images/runtime_method2.png)

runtime 处理method相关方法

![](/images/runtime_method3.png)

方法的invoke    objc_msgSend(receiver, selector, arg1, arg2, …)

typedef struct objc_selector *SEL;a Selector is the name of the method used to identity 方法的签名ID
比如 setObject:forKey:    setName:age:  这些形式的字符串就代表了sel的标识，但根据这些不知道这个方法参数的类型是什么
SEL aSelector = @selector(doSomething:) or SEL aSelector = NSSelectorFromString(@"doSomething:")

![](/images/runtime_method4.png)

method_types  方法的参数类型和返回值类型   比如方法setName:age:的参数是这样 v16@0:4@8i12    那些数字可能代表地址的偏移量

![](/images/runtime_method5.png)


![](/images/runtime_method6.png)

Implement -> id (*IMP)(id, SEL, …)  函数指针变量，指向方法实现代码块的入口地址

Implementation - the actual executable code block of a method.  it's a function pointer (an IMP).

a selector is like a key in in a hash table(NSDictionary) , and the value is the IMP of the method
class的方法列表其实是一个字典，key为selectors，IMPs为value。一个IMP是指向方法在内存中的实现

![](/images/runtime_method7.png)

利用runtime提供的method系列方法，可以很明显的知道可以将两个方法的实现exchange，这就是所谓的`swizzle`

```objc
swizzle : swizzling allows you to replace a method in an existing class with one of your own making . This approach can lead to a lot of unexpected behavior ,so it should be used very sparingly.
```

![](/images/runtime_method8.png)



