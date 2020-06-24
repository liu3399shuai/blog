---
title: "热修复"
date: 2018-07-22 11:31:32 +0800
comments: true
tags:
- 热修复
- JSPatch
categories:
- [热修复]
- [JSPatch]
---

介绍下ios平台实现的热修复

<!-- more -->

### 概念解释

* 热修复(也称热补丁、热修复补丁 英文 : hotfix)
  
  `(通过接口)动态下发代码(文件)，通过热修复引擎运行下发的代码，达到更改运行中的程序结构`

* 应用场景
  1. `时效性需求`，即a、不发版的情况下实时修复bug,用户无感知；b、及时应对一些来自各方面的不确定性情况
  2. `控制线上代码需求`，比如a、绕开apple审核，做apple不让做的内容；b、一些特殊需要，调用apple禁止的API

* 潜在危险
  1. 程序不可控(因为通过热修复可以调用和替换任意 OC 方法)
  2. 生态不安全

* 业界方案
![](/images/hotfix_1.png)

### 使用流程

server服务器端使用流程
![](/images/hotfix_2.png)

app端使用流程
![](/images/hotfix_3.png)

### 原理概述

#### Objective-C是动态语言，通过Objective-C Runtime在运行时可以进行如下操作

* 对类class可以进行`查找类`、`添加类`、`删除类`、`给类添加变量`、`给类添加方法`、`类名字符串反射到类` 等操作，也就是说 `通过字符串可以操作类`

	比如如下API都是操作类的方法：`super_class`、`class_name`、`class_var_list`、`class_method_list`、`class_protocol_list`、`class_info`、`class_isa`

* 对变量variable可以进行`添加变量`、`获取变量类型/值`、`更改变量类型/值`、`查找变量`、`变量名字符串反射到变量` 等操作，也就是说 `通过字符串可以操作变量`

	比如如下API都是操作变量的方法：`var_name`、`var_value`、`var_type`

* 对方法method可以进行`查找方法`、`获取方法`、`添加方法`、`判断方法是否存在`、`更改、替换方法实现hook`、`方法名字符串反射到方法` 等操作，也就是说 `通过字符串可以操作方法`

   比如如下API都是操作方法的方法：`method_name`、`method_imp`、`method_type`

* 对方法method进行调用,可从`target`、`method`、`argumentValue、returnValue`等方法组成部分进行拼装，可以使用
  1. 显示调用 []
  2. 隐士调用，如`performSelector`、`NSInvocation`、`objc_msgSend()`
  
  通过以上调用操作，也就是说 `通过字符串可以操作调用方法`

* 对方法进行消息转发操作

![](/images/hotfix_4.png)

通过字符串按照一定的约定规则，进行识别转换，可以调用和替换OC方法，达到更改程序结构的目的

#### JavaScript是解释型脚本语言，通过解释运行，边解释边执行，无需预先编译

* 解释型：把字符串当代码来执行
JavaScriptCore是一种JavaScript引擎，提供解释器环境
能够“读懂”JavaScript代码，对 JavaScript 代码进行解释执行

	主要为webkit提供脚本处理能力，可以将JavaScript的能力更轻便地、高性能地带给原生的iOS应用
可以实现OC和JavaScript两种语言的互转

```
•	JSContext     		为JavaScript提供运行环境 
•	JSValue        		是JavaScript和Object-C之间互换的桥梁,方法调用和参数数据互相转换都是它
•	JSManagedValue    	为开发人员提供对象内存管理 
•	JSVirtualMachine  	为JavaScript的运行提供了底层资源 
•	JSExport         	作为JavaScript和Objective-C两种语言的互通协议 
```

* 通过JavaScriptCore，Objective-C和JavaScript可以实现bridge通信，包括
	1. 参数转换
	![](/images/hotfix_5.png)
	2. 方法互通互调
	即，oc实现的方法，可以通过js去调用；js实现的方法，可以通过oc去调用

### 源码分析

![](/images/hotfix-001.png)
![](/images/hotfix-002.png)
![](/images/hotfix-003.png)
![](/images/hotfix-004.png)
![](/images/hotfix-005.png)
![](/images/hotfix-006.png)
![](/images/hotfix-007.png)
![](/images/hotfix-008.png)
![](/images/hotfix-009.png)
![](/images/hotfix-010.png)






