---
layout: post
title: "runtime运行时之类(class)"
date: 2016-02-29 18:55:14 +0800
comments: true
tags:
- [class]
categories:
- [runtime]
- [OC]
---

struct 结构体，构成了 数据结构的核心
面向对象就是造出了个class，class 类

<!-- more -->

### runtime使用之class

struct 结构体，构成了 数据结构的核心
面向对象就是造出了个class，class 类 就是一个指向结构体的指针变量，类里面的方法变量等都存放到了结构体里面的成员成员变量中
链表把数据串起来，形成一个组织


![](/images/runtime_class1.png)


class 里面主要由 变量、属性、方法、协议、类别

![](/images/runtime_class2.png)

runtime 可以实现 对class的add set get
对class里面的ivars 的add set get
对class里面的method的add set get
对class里面的protocol的add set get

Class 里面的API

![](/images/runtime_class3.png)


NSObject 里面有 - (Class)class  这是对象的方法，一般使用[self class],这样获取到的是self的指向的类，比如类A继承与类B，那么如果类A的viewDidAppear方法触发了，在类B的viewDidAppear的方法中打印[self class]，是类A的名字
+ (Class)class   这个就是类的方法了。

class_getInstanceMethod 类的实例化方法  --减方法
class_getClassMethod 类的方法 +加方法

![](/images/runtime_class4.png)

add class

![](/images/runtime_class5.png)

object 之instance

![](/images/runtime_class6.png)
