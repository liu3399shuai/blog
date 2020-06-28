---
title: "app无痕埋点"
date: 2020-1-11 20:31:00 +0800
comments: true
tags:
- 无痕埋点
categories:
- [无痕埋点]
---

介绍下app客户端实现的无痕埋点的实现方案

<!-- more -->

###无痕埋点实现指标

1. APP启动指标 

2. 页面浏览指标

3. 元素点击指标

4. 元素曝光指标

###无痕埋点实现厂商

![](/images/track-1.png)

###无痕埋点实现方案

1. 采集原始数据

利用oc的动态语言特性，runtime来实现关键方法API的hook，面向aop编程

* APP启动(热启动、冷启动)数据采集
	
```objc
+(void)load
{
    [NSNotificationCenter.defaultCenter addObserver:self selector:@selector(coldLaunchingNoti:) name:UIApplicationDidFinishLaunchingNotification object:nil];
    [NSNotificationCenter.defaultCenter addObserver:self selector:@selector(hotLaunchingNoti:) name:UIApplicationWillEnterForegroundNotification object:nil];
}

// 冷启动：打开APP
+(void)coldLaunchingNoti:(NSNotification *)noti
{
	// todo 
	// 记录record
}

// 热启动：APP从后台近前台
+(void)hotLaunchingNoti:(NSNotification *)noti
{
	// todo 
	// 记录record
}

```

* 页面浏览数据采集

```objc
+(void)load
{
    [self swizzleInstanceMethodWithOriginSel:@selector(viewDidAppear:) swizzledSel:@selector(tracklog_viewDidAppear:)];
    [self swizzleInstanceMethodWithOriginSel:@selector(viewDidDisappear:) swizzledSel:@selector(tracklog_viewDidDisappear:)];
}

- (void)tracklog_viewDidAppear:(BOOL)animated
{
    [self tracklog_viewDidAppear:animated];
    
	// todo 获取 pagename、pagetitle 
	
	// 记录record
}

- (void)tracklog_viewDidDisappear:(BOOL)animated
{
    [self tracklog_viewDidDisappear:animated];
    
	// todo 获取 pagename、pagetitle 
	
	// 记录record
}
```

* 元素点击数据采集

```objc
+(void)load
{
    [self swizzleInstanceMethodWithOriginSel:@selector(sendEvent:) swizzledSel:@selector(tracklog_sendEvent:)];
}

- (void)tracklog_sendEvent:(UIEvent *)event
{
    [self tracklog_sendEvent:event];
    
    // 找到点击的view
    
    // 获取该view的text、xpath、所在页面pagename、pagetitle、子children的信息
    
    // 记录 record
}
```

* 元素曝光数据采集

以UIScrollView为例

```objc
+(void)load
{
    [self swizzleInstanceMethodWithOriginSel:@selector(layoutSubviews) swizzledSel:@selector(tracklog_layoutSubviews)];
}

-(void)tracklog_layoutSubviews
{
    [self tracklog_layoutSubviews];
    
    CGRect visibleRect = CGRectMake(self.contentOffset.x, self.contentOffset.y, self.frame.size.width, self.frame.size.height);
    
    // 逐个比较，看该view是否在可视区域，如果在，dictionary中记录view可见性为1，并记录record
    for (int i = 0; i < self.userEnableSubViewTree.count; i++) {
        NSNumber *value = [self.tracklogDisplayMark objectForKey:item];
        BOOL contains = CGRectContainsRect(visibleRect, itemRect);
        BOOL intersects = CGRectIntersectsRect(visibleRect, itemRect);
        
        if (contains) {

			  // 记录 record
            
            [self.tracklogDisplayMark setObject:@(1) forKey:key];
        }
    }
}

-(void)viewDidLayoutSubviewsNoti:(NSNotification *)noti
{
	// 逐个遍历所有子view， 用一个dictionary记录该view的可见性，初始化时候可见效默认为0
	// 简略代码如下
    for (int i = 0; i < self.userEnableSubViewTree.count; i++) {
        self.tracklogDisplayMark setObject:@(0) forKey: item];
    }
}

```

2. 原始数据入库

将上面的环节采集到的数据，写入sqlite，并表明事件类型，时间、平台，用户、版本、等等 手机环境信息、app 信息等都记录下来

3. 上传数据

根据需要，可以实时上传，也可以 攒够10条上传，可以打开app时候上传，也可以进后台时候上传

需要密上传










