---
title: "iOS模块化去耦合实现"
date: 2017-01-14 15:33:31 +0800
comments: true
tags:
- 模块化
- 去耦合
categories:
- [模块化]
- [去耦合]
---

简单介绍下ios开发过程中实现的模块化去耦合，随着业务增长，这个是需要实现的

<!-- more -->

###模块介绍
  
  根据业务功能，拆分的相对独立的业务板块，比如 支付模块、 common模块、账号模块、产品模块、交易模块、运营模块、资产OK

###模块去耦合
	
  就是模块间没有直接的互相import，比如 产品模块，有个按钮，点击需要判断，如果登陆用户，则去购买页面，否则去登陆页面；这种情况下，如果不做去耦合，那么产品模块的controller需要持有购买页面的controller.h、登陆页面的controller.h、是否登陆的model.h ，这样就相当于 产品模块、账号模块、交易模块都重度耦合了，去耦合就是实现 产品模块里面 不持有 其它业务模块的class，达到即便其他业务模块的代码从工程remove掉，也可以 build success的效果
  
###实现方式

* URL scheme，open URL 方式，缺点是只能单向传递，反向回馈处理不友好

* 模块通信路由器利用protocol协议方式,runtime映射,面向接口编程POP，只声明方法，不实现方法，以接口依赖的方式取代对象依赖，双向通信，可以提供所有需求服务
  
  1. 简单介绍下路由器实现，2个API，一个是 负责register module，一个是负责获取module的instance
  
```objc
@interface ModuleServiceCenter : NSObject

- (void)registerService:(Protocol *)service forProvider:(Class)provider;

- (id)getProviderByService:(Protocol *)service;

@end

@implementation ModuleServiceCenter

- (void)registerService:(Protocol *)service forProvider:(Class)provider
{
    NSString *serviceStr = [NSString stringWithCString:protocol_getName(service) encoding:NSUTF8StringEncoding];
    
    NSString *providerStr = [NSString stringWithCString:class_getName(provider) encoding:NSUTF8StringEncoding];
    
    [_serviceProviderMap setObject:providerStr forKey:serviceStr];
}

- (id)getProviderByService:(Protocol *)service
{
    Class class = [self getClassByService:service];
    
    SEL sel = @selector(getModuleInstance);
    
    Method method = class_getClassMethod(class, sel);
    
    if (method) {
        id (*typed_msgSend)(id, SEL) = (void *)objc_msgSend;
        return typed_msgSend(class, sel);
    }
    
    return class;
}

@end

```
  
2. 接入方面，每个业务模块都有一份protocol文件，该文件代表了本模块能提供的所有的服务，比如如下是account模块的protocol文件,提供了5个API服务

```objc
@protocol AccountServiceProtocol

- (UIViewController *)getSettingController;

- (id)getLoginViewController;

- (void)refreshUserBaseInfo;

-(BOOL)isLogin;
-(BOOL)isBankCard;

@end

```

同时，业务模块还要有一份能实现业务模块protocol的provider类，比如

```objc
@interface AccountServiceProvider : NSObject <AccountServiceProtocol>

@end

@implementation AccountServiceProvider

- (UIViewController *)getSettingController;
{
    SettingViewController *vc = [[SettingViewController alloc] init];
    return vc;
}

- (id)getLoginViewController
{
    LoginViewController *vc = [[LoginViewController alloc] init];
    
    return vc;
}

- (void)refreshUserBaseInfo
{
    [[UserService service] refreshInfo];
}

-(BOOL)isLogin
{
    return UserInfo.share.login;
}

-(BOOL)isBankCard
{
    return isValid(UserInfo.share.bankName);
}

@end

```

3. 这样在+load方法里，对 account模块进行 register，注册完成后，其他业务模块就可以利用protocol路由来使用account模块提供的上述5个API

```objc
+ (void)load
{
    [[ModuleServiceCenter shareCenter] registerService:@protocol(AccountServiceProtocol) forProvider:[AccountServiceProvider class]];
}
```

4. 使用API是如下方式

```OBJC
// 在产品模块里
id<AccountServiceProtocol> accountProvider = [ModuleServiceCenter.shareCenter getProviderByService:@protocol(AccountServiceProtocol)];

if (!accountProvider.isLogin) {
	UIViewController *vc = [accountProvider getLoginViewController];
	[self.navigationController presentViewController:vc animated:YES completion:nil];
}
```

end，经过这个protocol router的方式，产品模块里面使用 account的服务，就可以不需要import account模块的类了，这样实现了代码逻辑去耦合





