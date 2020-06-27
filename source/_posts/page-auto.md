---
title: "页面布局自动化、页面跳转自动化"
date: 2017-03-04 16:32:34 +0800
comments: true
tags:
- 页面布局自动化
- 页面跳转自动化
categories:
- [页面布局自动化]
- [页面跳转自动化]
---

简单介绍下ios开发过程中实现的页面布局自动化、页面跳转自动化

<!-- more -->

* VIPER页面布局自动化

	通过预定义的json实现，可参考  https://github.com/andyyhope/nemo
	
	轻量级的可以参考下https://wiki.open.qq.com/index.php?title=mobile/SDK%E4%B8%8B%E8%BD%BD&oldid=47795
	
	都是通过json的配置，来进行ui的布局，需要app预埋支持
	
* 页面跳转自动化

	页面跳转都可以通过 scheme://的方式，也就是router方式来实现，包括参数的传递都在scheme://的query部分，比如 izrb://www.domain.com/product?pid=10，页面scheme配置好后，跳转可通过路由URLSchemeCenter来实现

以下为参考实现

```objc

typedef void (^CodeBlock)(id para);

@interface URLSchemeCenter : NSObject

- (BOOL)registerURL:(NSURL *)url forCodeBlock:(CodeBlock)codeBlock;

- (BOOL)executeURL:(NSURL *)url;

@end

@implementation URLSchemeCenter

- (instancetype)init
{
    self = [super init];
    
    if (self) {
        _selMap = [[NSMutableDictionary alloc] init];
        
        self.schemeMap = [[NSMutableDictionary alloc] init];
    }
    
    return self;
}

- (BOOL)registerURL:(NSURL *)url forCodeBlock:(CodeBlock)codeBlock
{
    if (![url isKindOfClass:NSURL.class]) {
        return NO;
    }
    
    if (!codeBlock) {
        return NO;
    }
    
    NSString *key = url.absoluteString.lowercaseString;
    CodeBlock value = [codeBlock copy];
    
    [self.schemeMap setObject:value forKey:key];
    
    return YES;
}

- (BOOL)executeURL:(NSURL *)url
{
    if (![url isKindOfClass:NSURL.class]) {
        return NO;
    }
    
    NSString *key = [url.absoluteString componentsSeparatedByString:@"?"].firstObject.lowercaseString;
    
    CodeBlock value = [self.schemeMap objectForKey:key];
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.1f * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSDictionary *params = [self getQueryObj:url.query];
        value(params);
    });
    
    return YES;
}

- (NSDictionary *)getQueryObj:(NSString *)string
{
    if (![string isKindOfClass: [NSString class]]) {
        return nil;
    }
    
    NSMutableDictionary *params = [NSMutableDictionary dictionaryWithCapacity:0];
    
    NSArray *queryStringPairs = [string componentsSeparatedByString:@"&"];
    
    [queryStringPairs enumerateObjectsUsingBlock:^(NSString *queryStringPair, NSUInteger idx, BOOL *stop) {
        NSRange range = [queryStringPair rangeOfString:@"="];
        if (range.location != NSNotFound) {
            NSString *key = [queryStringPair substringToIndex:range.location];
            NSString *value = [queryStringPair substringFromIndex:range.location + 1];
            [params setValue:value forKey:key];
        }
    }];
    
    return params;
}

@end

```
