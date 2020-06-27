---
title: "iOS framework 制作"
date: 2017-01-15 12:30:20 +0800
comments: true
tags:
- framework
- static library
categories:
- [framework]
- [static library]
---

简单介绍下ios开发过程中framework的制作

<!-- more -->

###概念解释

framework 路径更优雅些，结构清晰(对library的封装，同时封装了对应的header文件、以及引用的资源文件(如果有))

framework也分为 静态framework 、动态framework

查看是静态还是动态 在终端里面 file framework就可以显示

.a 文件 即 static library 是经过 compile linked 后的可执行代码，链接时完整地拷贝至可执行文件中，被多次使用就有多份冗余拷贝

###制作流程简要记录

1. Xcode new一个 framework类型的的 project

2. project下新建podfile，配置好podfile后，进行pod install (podfile 里面使用 use_frameworks!)

3. install完成后，workspace下的目标target配置更改(Mach-O Type要选择Dynamic Library(也可以使用 Static Library 看需求，最好是 Static Library )、Buld Setting中Other Linker Flags要加上-all_load、Buld Setting中Build Active Architecture Only选No)
查看是否是动态库、静态库参考 https://www.jianshu.com/p/77343def4574

4. 目标target配置支持bitcode(Enable Bitcode = Yes、Other C Flags中添加-fembed-bitcode、Other Linker Flags 中加入 -fembed-bitcode)

5. workspace里面的scheme选项里面把目标scheme new出来，点击Edit Scheme 把Run Build 等命令的Build Configuration 都改成 Release ，完后进行进行build

6. product文件夹下的目标framework既是结果，此时需要此framework同时支持模拟器和真机，所以需要对 device 和 simulator 分别 build一次，将两次产生的framework合并，查看是否支持真机、模拟器 ，以及合并命令

