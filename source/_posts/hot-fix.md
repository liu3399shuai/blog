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

### 使用流程
可以通过访问https://www.app-domain.com/apple-app-site-association来了解目前拦截了哪些WAP页面
如果需要访问WAP页面，可以在点击链接打开APP后，在屏幕的右上角会有个网页进入选项，点击即可进入WAP浏览模式（此操作为记忆模式，下次点开链接会在WAP中访问）。
如果需要点击链接访问APP，可以下拉页面顶部可见一个操作条，点击操作条中的"打开"即可重新跳转至APP中。

例如，知乎app的支持的deeplink为<https://www.zhihu.com/apple-app-site-association>

### 原理概述
APP端需要做域名关联和URL的解析，即可完成通用链接的配置和使用。
工程配置下支持Associated Domains

```objc
// deeplink在app里面的回调
-(BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler{

    NSLog(@"userActivity : %@",userActivity.webpageURL.description);
    return YES;
}
```

### 源码分析





