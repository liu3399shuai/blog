---
layout: post
title: "自定义键盘和输入框"
date: 2016-06-07 18:02:04 +0800
comments: true
tags:
- [UITextField]
- [inputView]
categories:
- [键盘]
- [OC]
---


为什么点击UITextField键盘就抬起来了呢？为什么点击键盘里的字符，会显示在UITextField输入框里面呢？如何自定义点击UITextField后抬起来的键盘,并将自定义键盘里面的被点击字符传入UITextField

<!-- more -->

##### 为什么点击UITextField键盘就抬起来了呢？

##### 为什么点击键盘里的字符，会显示在UITextField输入框里面呢？

##### 如何自定义点击UITextField后抬起来的键盘,并将自定义键盘里面的被点击字符传入UITextField

##### 如何自定义一个view，来模拟UITextField的功能？就是说点击这个view，键盘抬起，点击键盘里面的字符，会显示到这个view上面



### 1 UITextField 继承自 UIResponder，UIResponder可以响应用户的交互

```
@interface UIResponder

- (BOOL)becomeFirstResponder;

- (BOOL)resignFirstResponder;

- (BOOL)isFirstResponder;

```


### 2 可以响应用户事件了，这还不够，UIButton也可以，点击它怎么不抬起键盘。UITextField有个属性 inputView，看解释

```
// Presented when object becomes first responder.  If set to nil, reverts to following responder chain.  If
// set while first responder, will not take effect until reloadInputViews is called.
@property (nullable, readwrite, strong) UIView *inputView;

```

也就是说，如果UITextField的这个属性是个nil，那点击它后，键盘也抬不起来了，也就是说如果重新给inputView赋值一个自定义的view，那点击UITextField后，这个自定义的view就会被抬起来了，这就是所谓的自定义的键盘，其实就是个view


### 3 点击UITextField后，键盘抬起来了，可是点击键盘里面的字符怎么会显示到UITextField里面的呢？

```
@interface UITextField : UIControl <UITextInput, NSCoding> 

@protocol UITextInput <UIKeyInput>
```

是因为UITextField遵守了UITextInput这个协议。而UITextInput又遵守了UIKeyInput这个协议

```
@protocol UIKeyInput <UITextInputTraits>

- (BOOL)hasText;
- (void)insertText:(NSString *)text;
- (void)deleteBackward;

@end
```

UIKeyInput这个协议里面的这三个方法，就是实现键盘里被点击字符在UITextField输入框里面显示的方法，也就是说，可以手动调用这三个方法，来实现自定义键盘的功能，如下

在自定义的键盘(就是一个view)上面放置很多btn，btn事件点击如下，这样没点击一个btn，UITextField里面就会做出相应的字符显示或删除

```
-(void)btnClick:(UIButton *)btn
{
    NSInteger tag = btn.tag;
    
    if (10 <= tag && tag <= 19) {
        [_textField insertText:[NSString stringWithFormat:@"%ld",tag - 10]];
    }else if (tag == 20){
        [_textField insertText:@"00"];
    }else if (tag == 21){
        [_textField insertText:@"."];
    }else if (tag == 100){
        [_textField deleteBackward];
    }else if (tag == 200){
        [_textField resignFirstResponder];
    }else if (tag == 300){
        [_textField resignFirstResponder];
    }else{
        [_textField resignFirstResponder];
    }
    
    [[UIDevice currentDevice] playInputClick];
}

```


## 4 刚才说的点击UITextField键盘抬起是因为有个inputView属性，那一个自定义的view，没有这个属性，点击后怎么把键盘抬起来？

这个也可以，没有inputView，但可以遵守这个协议`<UIKeyInput>` 也可以让键盘抬起来

```

@interface PswTextView : UIView <UITextInput> 

```

点击PswTextView后，它becomeFirstResponder，键盘被顺利抬起,可是如何捕获键盘里面的被点击字符呢？如何显示呢？

重写协议里面的以下方法

```
_textStore 是PswTextView里面的全局可变字符串变量，用来存储当前键盘里面记录的点击字符

- (void)deleteBackward
{
    if (_textStore.length == 0) {
        return;
    }
    
    NSRange theRange = NSMakeRange(_textStore.length - 1, 1);
    [_textStore deleteCharactersInRange:theRange];
    
    [self setNeedsDisplay];
}

- (void)insertText:(NSString *)text
{
    if (_textStore.length == 6) {
        return;
    }
    
    if (![text isPureNumber]) {
        return;
    }
    
    [_textStore appendString:text];
    
    [self setNeedsDisplay];
}

- (BOOL)hasText
{
    return (_textStore.length > 0);
}

```

重写UIResponder里面的方法

```
- (BOOL)canBecomeFirstResponder
{
    return YES;
}

- (BOOL)becomeFirstResponder
{
    [_textStore setString:@""];
    [self setNeedsDisplay];
    
    return [super becomeFirstResponder];
}
```

在这个view上面显示的键盘被点击字符的话，就用到`drawRect `方法了，在键盘里面的字符变化时候调用一句`setNeedsDisplay `

```
-(void)drawRect:(CGRect)rect
{    
    [_textStore drawInRect:rect withAttributes:@{NSForegroundColorAttributeName:self.color,NSFontAttributeName:self.font}];
}
```


## 自定义键盘、自定义输入框详细代码 请看[github](https://github.com/liu3399shuai/keyboard-Input)
