---
title: "iOS安全攻防之加固"
date: 2018-10-00 18:00:00 +0800
comments: true
tags:
- 加固
- 安全攻防
categories:
- [加固]
- [安全攻防]
---

简单介绍下iOS的安全攻防之加固

<!-- more -->

### 加固

加固 和  代码混淆 是一回事，都是 防止 别人轻易的破解APP里面的内容，降低可读性，目的就是防止别人用class_dump等工具把代码明文给还原出来

用这些加固方式 也可以 解决 apple审核时候遇到的4.3马甲包问题

* 主要有5类加固维度

1. 字符串加密
	
	![](/images/strong-1.png)
	
2. 符号混淆(类、方法、变量 名称混淆)
	
	![](/images/strong-2.png)
	
3. 逻辑混淆(指令混淆)
	
	逻辑混淆(指令混淆)：通过将原始代码的控制流进行切分、打乱、隐藏，或在函数中插入花指令来实现对代码的混淆，使代码逻辑复杂化但不影响原始代码逻辑。
	
	![](/images/strong-3.png)
	
4. 代码虚拟化

5. 防调试

* 加固方式

1. 源码加固( 编译前，在源工程里面 加入 混淆工具脚步等，执行build)

2. 二进制加固(编译后，将 应用程序 .app 传入至加固平台，输出 加固后的 .app)

* 加固开源厂商

obfuscator-llvm  ： https://github.com/obfuscator-llvm/obfuscator/tree/llvm-4.0

class-guard  ： https://github.com/Polidea/ios-class-guard

polidea / sirius  :  https://github.com/Polidea/SiriusObfuscator

https://github.com/lyzz0612/iosMixTools

https://www.jianshu.com/p/a631b5584de6


