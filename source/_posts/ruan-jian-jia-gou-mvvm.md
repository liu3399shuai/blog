---
layout: post
title: "软件设计层次架构"
date: 2016-03-08 18:33:52 +0800
comments: true
tags:
- MVVM
categories:
- [软件架构]
- [架构]
---

软件层次

`工具层`(utility，通用的工具，和具体的业务无关)

`网络层`(负责和server进行交互)

<!-- more -->

### 软件层次

`工具层`(utility，通用的工具，和具体的业务无关)

`网络层`(负责和server进行交互)

`数据持久层`(本地缓存、数据库)

`model层`(原始数据的类载体，比如来自网络或数据库的时间戳，不进行处数据理)

`逻辑层`(viewmodel层，软件业务功能逻辑相关，可以给每个界面增加个逻辑层，也可以给一个业务功能多个界面增加一个逻辑层)

`UI层`(显示的view)

`控制层`(controller是个大杂炖，上面那些层都在这个controller里面彼此交互)

### 软件框架

##### 大多是`MVC`模式

![](/images/mvc.png)

缺点：经常会导致C很臃肿，不易管理，所以出现了MVVM。

##### MVVM模式

`MVVM`就是将业务逻辑从controller里面抽出来单独生成一个类，降低耦合

如果要想把业务逻辑从controller中抽取出来，以实现逻辑层(viewmodel)是逻辑的载体，那么逻辑层至少需要

```
1 实时的接受view里面各种数据源和事件源的变化(比如输入框、按钮等)
2 逻辑任务处理完成后将数据、动作输出至view/controller
```

要实现这两条，就需要`响应式编程`的概念。

```
函数响应式编程(Functional Reactive Programing : FRP)  :  面向数据流的变化传播(管道处理，比如过滤，类型映射，订阅，组合,属性绑定)
```

比如 excel 3-8数码管 订阅(公众号、日报),详见[wiki](https://zh.wikipedia.org/wiki/%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B)

数据流，逻辑流 这种概念思想很重要,和命令式编程大不一样

由此，`Reactive Cocoa` 应运而生。

Reactive Cocoa 可以承担以下任务

1 数据流统一封装分发

![](/images/mvvm3.png)

2 数据(双向)绑定

![](/images/mvvm1.png)

3 动作事件通知

![](/images/mvvm2.png)

代码实例

```
// data binding
RAC(_viewModel,phone) = _phoneTF.rac_textSignal; // 这样可以(只能)观察到textField 源于键盘输入导致的值变化
RAC(_phoneTF,text) = [RACObserve(_viewModel, phone) distinctUntilChanged];

// 清空_phoneTF
_viewModel.phone = @""; // 代码实现
// _phoneTF.text = nil; // 这种方式不行
```

关于使用RAC(...)，有时会有这样的错误`error : is already bound to key path "" on object  rac`，原因是

```
RAC(self.viewModel,smsCode) = _phoneTF.rac_textSignal;
RAC(self.viewModel,smsCode) = _codeTF.rac_textSignal;
给smsCode只能绑定一次,绑给别的信号了，就不能再绑给另外一个信号了,所以绑定慎用，但可以subscribeNext啊
```

[RACSignal createSignal] 方式创建signal，只创建一次就OK

```
-(RACSignal *)validatePhoneSignal
{
    if (!_validatePhoneSignal) {
        _validatePhoneSignal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            
            if (isValid([[UserBaseInfo share] identityName]) && _idCode.length != 15 && _idCode.length != 18) {
                [subscriber sendError:Error(errCode_toast, @"请输入正确的15或18位身份证号")];
                return nil;
            }
            
            [_service verifyCode:_smsCode idCard:_idCode finish:^(NSError *error) {
                
                if (error) {
                    [subscriber sendError:error];
                }else{
                    
                    [subscriber sendNext:@(YES)];
                    [subscriber sendCompleted];
                }
            }];
            
            return nil;
        }];
    }
    
    return _validatePhoneSignal;
}
```

[RACSubject subject] 创建的signal，每次都得创建

```
-(RACSignal *)rechargeSignal
{    
    RACReplaySubject *subject = [RACReplaySubject subject];
    _rechargeSignal = [subject deliverOnMainThread];
    
    [self.submitSignal subscribeNext:^(NSNumber *x) {
        
        if (x.boolValue) {
            [subject sendNext:@(YES)];
            [subject sendCompleted];
        }else{
            [self.querySignal subscribeNext:^(id x) {
                [subject sendNext:@(YES)];
                [subject sendCompleted];
            } error:^(NSError *error) {
                [subject sendError:error];
            }];
        }
        
    } error:^(NSError *error) {
        [subject sendError:error];
    }];
            
    return _rechargeSignal;
}
```


