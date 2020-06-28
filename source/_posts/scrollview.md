---
title: "UIScrollView嵌套联动效果"
date: 2019-12-03 17:06:00 +0800
comments: true
tags:
- UIScrollView嵌套联动效果
categories:
- [UIScrollView嵌套联动效果]
---

介绍下app iOS的UIScrollView嵌套联动效果

<!-- more -->

###UIScrollView嵌套联动效果

UIScrollView嵌套 + Tab吸顶 联动效果

能够保证 1、竖直方向联动效果很自然，不卡顿 2、tab内容区的左右切换很自然，并且不会影响到上下滑动，也就是 左右滑动和上下滑动能够做到互斥，不会同时发生

看下图示效果

![](/images/scroll-1.jpg)

###解决思路

整个上下滑动是个UIScrollView，tab分页的内容区是个UIScrollView，里面是UITableView，监听偏移量，初始化时，父UIScrollView可以滑动，子UIScrollView不可以滑动，当父UIScrollView上滑至可以吸顶位置时，不让它滑动，让子UIScrollView可以滑动

###解决方案

手势穿透 + 响应链，这样在子UIScrollView上的手势可以传递到父UIScrollView，达到共享手势

内层的 UITableView（TAB 栏里面）和外层的 UIScrollView 同时响应用户的手势滑动事件

父UIScrollView实现UIGestureRecognizerDelegate的代理方法

```objc

- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
{
  //在该方法中，判断otherGestureRecognizer.view如果为需要手势穿透的控件，则返回yes，否则返回no
}

父UIScrollView的hitTest方法返回值为 子UIScrollView
- (UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event
{

}

```

代码实现

```objc
父UIScrollView

1、遵守协议 <UIGestureRecognizerDelegate>

- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
{
    if ([self.simultaneouslyScrollList containsObject:otherGestureRecognizer.view]) {
        return YES;
    }
    
    return NO;
}

2 - (void)scrollViewDidScroll:(UIScrollView *)scrollView中重置contentOffset

if (self.scrollEnable) {
                if (self.contentOffset.y > self.maxY) {
                    
                    self.scrollEnable = 0;
                    
                    self.contentOffset = CGPointMake(self.lockDirectionDefaultVaule, self.maxY);
                }else{
                    self.contentOffset = CGPointMake(self.lockDirectionDefaultVaule, self.contentOffset.y);
                }
            }else{
                self.contentOffset = CGPointMake(self.lockDirectionDefaultVaule, self.maxY);
            }
```

```objc
子UITableView

if (self.scrollEnable) {
            if (scrollView.contentOffset.y < 0) {
                self.scrollEnable = 0;


                scrollView.contentOffset = CGPointMake(scrollView.contentOffset.x, 0);


                if (superScrollBlock) {
                    superScrollBlock(1);
                }
            }else{
                if (superScrollBlock) {
                    superScrollBlock(0);
                }
            }
        }else{
            scrollView.contentOffset = CGPointMake(scrollView.contentOffset.x, 0);


            if (superScrollBlock) {
                superScrollBlock(1);
            }
        }
```

###疑难点

1. tab里面有的tableview或者scrollview里面嵌套webview，在滑动过程中特别容易导致卡顿，一顿一顿的，当去掉webview后，卡顿效果没有了，但是产品上要求联动效果里面必须支持webview，后来通过调试发现有webview，但是webview的userInteractionEnabled为0也不会导致页面卡顿，所以解决方案就是，只允许当前tab下的tableview的userInteractionEnabled为1，其他tab下的tableview的userInteractionEnabled为0

2. 当tab的左右滑动时候，经常会导致主scrollview的上下滑动，水平滑动和竖直滑动没有互斥
如何判断有偏斜角度的手势为 水平手势，还是竖直手势，检测 目的滑动方向
因为父scrollview是允许手势穿透的，所以它里面的子scrolview的滑动都会传递到父scrollview，导致多个scrollview同时滑动的现象，原则上，同一时间只允许有一个scrollview可以滑动，不管是水平还是竖直，所以对父scrollview的contentInset设置为UIEdgeInsetsMake(1, 0, 0, 1); 这样能保证scrollview既能支持水平滑动，也能支持竖直滑动，(尽管实际上它应该只能竖直滑动)，这样的话，在手势代理方法gestureRecognizerShouldBegin中就能拿到 当前手势的 水平速度、竖直速度分别是多少，代码判断 水平速度>竖直速度 就为 水平滑动，反之为竖直滑动，如果父scrollview为竖直滑动效果，那么当手势为水平滑动时候，在scrolldidscroll代理方法中，重置scrollview的contentoffset，保证父scrollview不产生水平滑动现象，同时因为手势被判断为水平滑动，父scrollview竖直方向上也不会滑动

这样处理后，原则上可以解决 水平和竖直滑动互斥，但是调试发现在gestureRecognizerShouldBegin中捕获手势速率经常为0 (主scrollvie在竖直方向已经滑动，有动画滑动，还未停止时候，再次有手势触发的时候会发生，tab左右滑动就会导致，两个方向同时滑动)，这种情况下，依然实现不了互斥，依然会发生 水平和竖直同时滑动现象

自己处理手势方向识别，并做出是否滑动，并没有系统实现的好，所以在是否允许手势传递方法中，判断，是子tableView的时候才允许

```objc
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
{
    if ([self.simultaneouslyScrollList containsObject:otherGestureRecognizer.view]) {
        return YES;
    }
    
    return NO;
}
```