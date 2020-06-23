---
layout: post
title: "runtime运行时的理解"
date: 2016-02-29 18:39:40 +0800
comments: true
tags:
- 运行时
categories:
- [runtime]
- [OC]
---

runtime运行时的理解：实现动态语言的关键

OC是一门动态语言，而实现动态语言的关键就是 runtime(运行时)

<!-- more -->

### runtime运行时的理解：实现动态语言的关键

OC是一门动态语言，而实现动态语言的关键就是 runtime(运行时)

动态语言：在运行时可以`动态操作程序`(动态的对class、method、variable、protocol进行add、set、get等操作)，比如get：运行时才去检查确认其结构(比如数组字典都可以是id类型)，也就是，尽可能把编译和链接时要执行的逻辑延迟到运行时
有一个运行时系统 (runtime system) 来执行编译后的代码(比如说数据类型的检查，编译时候可以随便给id类型转换赋值，都不会报错，直到运行时needed时候才会报错，再比如 方法的调用，[self performSelector],如果selector没有可执行的指针地址(implement)，编译时候不会error，而是到运行时候，使用的时候，再现去找selector的指针地址，找不多则提供消息转发机制，否则crash，再比如，可以在编译链接完成后，根据实际需要动态的对class、variable、method、protocol进行add、set、get等操作)

静态语言：在编译时就做了所有的检查和命令（数据类型检查、每个方法名对应的方法执行的入口地址检查等等）
静态语言里面的编译器其中最最基础和原始的目标之一就是把一份代码里的函数名称，转化成一个相对内存地址，把调用这个函数的语句转换成一个jmp跳转指令。在程序开始运行时候，调用语句可以正确跳转到对应的函数地址。 这样很好，也很直白，但是。。。太死板了。everything is per-determined

一句话解释静态语言和动态语言 Static typing when possible, dynamic typing when needed

实际使用runtime动态调用的地方 举例：`respondsToSelector`、`performSelector`、`isKindOfClass`、`conformsToProtocol`

动态语言好处：编写代码灵活方便，缺点：不易调试。。。比如JavaScript，Python，Ruby都是动态语言
静态语言好处：结构规范、方便调试，缺点：代码编写不灵活。。不如C++、Java都是静态语言

runtime 参考
<https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/>
<http://www.opensource.apple.com/source/objc4/>
