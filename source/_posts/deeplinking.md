---
title: "deeplinking(通用链接)"
date: 2018-03-21 9:31:20 +0800
comments: true
tags:
- deeplinking
- 通用链接
categories:
- [deeplinking]
- [通用链接]
---

简单介绍下ios的deeplinking(通用链接)

<!-- more -->

## 前沿
经常看到从Safari里面点击一个链接 直接跳转到指定APP，这个现象实现的方式就是使用deeplinking技术

通用链接技术可以实现在安装APP的前提下，点击WAP页的相关链接，可以直接打开APP（通过APP代码的控制可以跳转到对应的视图）

### 链接的控制
可以通过访问https://www.app-domain.com/apple-app-site-association来了解目前拦截了哪些WAP页面
如果需要访问WAP页面，可以在点击链接打开APP后，在屏幕的右上角会有个网页进入选项，点击即可进入WAP浏览模式（此操作为记忆模式，下次点开链接会在WAP中访问）。
如果需要点击链接访问APP，可以下拉页面顶部可见一个操作条，点击操作条中的"打开"即可重新跳转至APP中。

例如，知乎app的支持的deeplink为<https://www.zhihu.com/apple-app-site-association>

### APP端配置
APP端需要做域名关联和URL的解析，即可完成通用链接的配置和使用。
工程配置下支持Associated Domains

```objc
// deeplink在app里面的回调
-(BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler{

    NSLog(@"userActivity : %@",userActivity.webpageURL.description);
    return YES;
}
```

关于通用链接的详细说明可参考官网：https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html