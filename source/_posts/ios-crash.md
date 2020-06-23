---
title: "iOS crash的相对全面的介绍"
date: 2019-04-23 10:24:00
comments: true
tags:
- iOS
- crash
categories:
- [crash]
- [sentry]
- [sdk]
---

本文介绍了iOS中的crash的分类、收集、存储、上传、符号化等流程

<!-- more -->

## crash分类

* crash分为 math exception 、 signal 、 NSException 三种类型，每一种类型crash都处在不同系统层级上
	1. 内核级异常 math exception
	2. 系统级异常 signal
	3. 应用级异常 NSException

## 收集crash

利用 NSSetUncaughtExceptionHandler() NSGetUncaughtExceptionHandler() 方法，去捕获crash信息

如果多平台收集上传，需要使用NSGetUncaughtExceptionHandler()，并顺序传递exception，如下

```objc
static NSUncaughtExceptionHandler *MisCrashManager_PreviousUncaughtExceptionHandler;

static void MisCrashManager_UncaughtExceptionHandler(NSException *exception)
{
    if(MisCrashManager_PreviousUncaughtExceptionHandler){
        MisCrashManager_PreviousUncaughtExceptionHandler(exception);
    }
    
    MisCrashManager_DealException(exception);
}

NSSetUncaughtExceptionHandler(&MisCrashManager_UncaughtExceptionHandler);
```

在收集的过程中，要多参数，多维度的去收集，包括不限于`base_address`、`slide_address`、`device_id`、`system_version`、`device_type`、`version`、`project`、`title`、`content`等参数，当然开发者需要根据本身业务去增加或减少

常用的收集库`PLCrashReporter`、`CrashKit`，可以引入进来

## 存储crash

将上一步收集的crash存储到本地，可以是sql的数据结构，也可以是xml、json等list数据结构，下一次app打开时候，实时的上传到服务器

如下事例为收集、存储

```objc
static void MisCrashManager_DealException(NSException *exception)
{
    NSString *title = [NSString stringWithFormat:@"%@ %@",exception.name,exception.reason];
    
    NSData *callStack = [NSJSONSerialization dataWithJSONObject:exception.callStackSymbols options:NSJSONWritingPrettyPrinted error:nil];
    NSString *callStackStr = [[NSString alloc] initWithData:callStack encoding:NSUTF8StringEncoding];
    
    NSDictionary *remark = @{@"base_address"    : MisCrashTool.baseAddress ? : @"",
                             @"slide_address"   : MisCrashTool.slideAddress ? : @"",
                             @"device_id"       : MisCrashManager.shared.deviceID ? : @""
                             };
    
    NSData *remarkData = [NSJSONSerialization dataWithJSONObject:remark options:NSJSONWritingPrettyPrinted error:nil];
    NSString *remarkStr = [[NSString alloc] initWithData:remarkData encoding:NSUTF8StringEncoding];
    
    NSDictionary *info = @{@"sequence_id"       : MisCrashManager.shared.userID ? : @"",
                           @"platform"          : @"ios",
                           @"system_version"    : UIDevice.currentDevice.systemVersion ? : @"",
                           @"device_type"       : MisCrashTool.deviceName ? : @"",
                           @"version"           : NSBundle.mainBundle.infoDictionary[@"CFBundleShortVersionString"] ? : @"",
                           @"project"           : MisCrashManager.shared.projectID ? : @"",
                           @"create_time"       : @(@(NSDate.date.timeIntervalSince1970*1000).longLongValue),
                           @"title"             : title ? : @"",
                           @"content"           : callStackStr ? : @"",
                           @"remark"            : remarkStr ? : @""
                           };
    NSData *infoData = [NSJSONSerialization dataWithJSONObject:info options:NSJSONWritingPrettyPrinted error:nil];
    NSString *infoStr = [[NSString alloc] initWithData:infoData encoding:NSUTF8StringEncoding];
    
    NSString *documentPaths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
    documentPaths = [documentPaths stringByAppendingPathComponent:@"mis_crash_exception.txt"];
    
    NSArray *oldList = [[NSArray alloc] initWithContentsOfFile:documentPaths];
    
    NSMutableArray *list = [[NSMutableArray alloc] initWithArray:oldList];
    [list addObject:infoStr ? : @""];
    [list writeToFile:documentPaths atomically:YES];
}
```

## 上传crash
在app下次打开时候，https实时的上传到服务器,上传完一条记得把本地存储的该条记录删除

## 符号化crash

导出dsym文件，需要使用atos、symbolicate等操作进行符号化，也可以使用dsymTools(https://github.com/answer-huang/dSYMTools)

符号化，拿着 符号地址 和 DSYM 进行 还原
symbol_address 是 stack_address 减去 slide_address 的结果

堆栈地址StackAddress - 加载地址LoadAddress = 符号地址SymbolAddress
0x1003f34a0 - 0xea4000 = symbol_address

atos -o xxx.app.DSYM/Contents/Resources/DWARF/xxx -arch arm64 symbol_address
atos -o xxx.app.DSYM/Contents/Resources/DWARF/xxx -arch arm64 0x1003F01FC

原始采集到的crash信息如下

```
CoreFoundation 0x0000000182eefd5c <redacted> + 52 
UIKit 0x000000018d25bc7c <redacted> + 1352
xxx      0x1003f34a0   xxx + 4142240

```

符号化后的明文为

```
3 UIKit -[UIWebSelectSinglePicker pickerView:didSelectRow:inComponent:] + 72
4 UIKit -[UIPickerTableView _scrollingFinished] + 188
5 xxx -[ServiceViewController tableView:didSelectRowAtIndexPath:] + 460
```

## 第三方优质服务商/开源服务

`apple crash`、`google firebase`、`bugly`、`sentry开源且支持私有化部署`

## sentry私有化部署接入记录

Client Key (dsn) 在sentry后台的`Settings -> Projects -> 目标project -> Client Keys -> DSN`

Auth Tokens 在sentry后台的`Settings -> Account -> Auth Tokens`, 使用sentry-cli命令行登录时候需要用到的token

DSYM 在在sentry后台的`Settings -> Projects -> 目标project -> Debug Files`

dsym符号表的上传需要使用sentry-cli命令，可在命令行手动上传，可写脚本嵌到工程的 Build Phases里面自动上传

安装sentry-cli : 命令行 `curl -sL https://sentry.io/get-cli/ | bash`

登录sentry-cli : `sentry-cli —url  http://sentry.xxxdomain.com/  login`



