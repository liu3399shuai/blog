---
title: "iOS安全攻防之破壳"
date: 2018-10-00 19:00:00 +0800
comments: true
tags:
- 破壳
- 安全攻防
categories:
- [破壳]
- [安全攻防]
---

简单介绍下iOS的安全攻防之破壳

<!-- more -->

### 使用class_dump

class_dump : 利用runtime特性来dump出工程的class信息，也就是还原出.h文件
class_dump 只是 逆向破壳的第一阶段，后面还很多

终端里面运行下载后的 class-dump 可执行文件

class_dump 只支持OC，不支持swift，有swift的会dump失败

在终端，使用class-dump对 IPA里面的 .app 执行文件 进行dump

![](/images/breakhull-1.png)

### 使用 IDA 或 Hopper Disassembler

反编译出来的伪代码  示例图如下，具体使用 可参考 加固厂商顶象加固报告

![](/images/breakhull-2.png)

```objc
                     -[AuthInfoController streetAction]:
00000001000b5b3c         sub        sp, sp, #0x70                               ; Objective C Implementation defined at 0x100bbded0 (instance method), Begin of try block, DATA XREF=0x100bbded0
00000001000b5b40         stp        x24, x23, [sp, #0x30]
00000001000b5b44         stp        x22, x21, [sp, #0x40]
00000001000b5b48         stp        x20, x19, [sp, #0x50]
00000001000b5b4c         stp        x29, x30, [sp, #0x60]
00000001000b5b50         add        x29, sp, #0x60
00000001000b5b54         mov        x20, x0
00000001000b5b58         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5b5c         ldr        x1, [x8, #0xa68]                            ; "resetFirstResponsder",@selector(resetFirstResponsder)
00000001000b5b60         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5b64         add        x0, sp, #0x28
00000001000b5b68         mov        x1, x20
00000001000b5b6c         bl        imp___stubs__objc_initWeak                  ; objc_initWeak
00000001000b5b70         adrp      x8, #0x100c6d000
00000001000b5b74         ldr        x0, [x8, #0xc8]                            ; objc_cls_ref_InfoSheetView,__objc_class_InfoSheetView_class
00000001000b5b78         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5b7c         ldr        x1, [x8, #0xa70]                            ; "instanceView",@selector(instanceView)
00000001000b5b80         bl        imp___stubs__objc_msgSend                   ; objc_msgSend, End of try block started at 0x1000b5b3c, Begin of try block (catch block at 0x1000b5d78)
00000001000b5b84         mov        x29, x29                                    ; End of try block started at 0x1000b5b80, Begin of try block
00000001000b5b88         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5b8c         mov        x19, x0
00000001000b5b90         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5b94         ldr        x21, [x8, #0xaf8]                          ; "streetDic",@selector(streetDic)
00000001000b5b98         mov        x0, x20                                     ; End of try block started at 0x1000b5b84, Begin of try block (catch block at 0x1000b5d78)
00000001000b5b9c         mov        x1, x21
00000001000b5ba0         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5ba4         mov        x29, x29                                    ; End of try block started at 0x1000b5b98, Begin of try block
00000001000b5ba8         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5bac         mov        x22, x0
00000001000b5bb0         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5bb4         ldr        x1, [x8, #0xa00]                            ; "allKeys",@selector(allKeys)
00000001000b5bb8         bl        imp___stubs__objc_msgSend                   ; objc_msgSend, End of try block started at 0x1000b5ba4, Begin of try block (catch block at 0x1000b5d78)
00000001000b5bbc         mov        x29, x29                                    ; End of try block started at 0x1000b5bb8, Begin of try block
00000001000b5bc0         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5bc4         mov        x23, x0
00000001000b5bc8         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5bcc         ldr        x1, [x8, #0xaa0]                            ; "descendingOrder",@selector(descendingOrder)
00000001000b5bd0         bl        imp___stubs__objc_msgSend                   ; objc_msgSend, End of try block started at 0x1000b5bbc, Begin of try block (catch block at 0x1000b5d78)
00000001000b5bd4         mov        x29, x29                                    ; End of try block started at 0x1000b5bd0, Begin of try block
00000001000b5bd8         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5bdc         mov        x24, x0
00000001000b5be0         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5be4         ldr        x1, [x8, #0xa80]                            ; "setContentData:",@selector(setContentData:)
00000001000b5be8         mov        x0, x19                                     ; End of try block started at 0x1000b5bd4, Begin of try block (catch block at 0x1000b5d78)
00000001000b5bec         mov        x2, x24
00000001000b5bf0         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5bf4         mov        x0, x24                                     ; End of try block started at 0x1000b5be8, Begin of try block
00000001000b5bf8         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5bfc         mov        x0, x23
00000001000b5c00         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5c04         mov        x0, x22
00000001000b5c08         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5c0c         mov        x0, x20                                     ; End of try block started at 0x1000b5bf4, Begin of try block (catch block at 0x1000b5d78)
00000001000b5c10         mov        x1, x21
00000001000b5c14         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5c18         mov        x29, x29                                    ; End of try block started at 0x1000b5c0c, Begin of try block
00000001000b5c1c         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5c20         mov        x20, x0
00000001000b5c24         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5c28         ldr        x1, [x8, #0xaa8]                            ; "setContentParseData:",@selector(setContentParseData:)
00000001000b5c2c         mov        x0, x19                                     ; End of try block started at 0x1000b5c18, Begin of try block (catch block at 0x1000b5d78)
00000001000b5c30         mov        x2, x20
00000001000b5c34         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5c38         mov        x0, x20                                     ; End of try block started at 0x1000b5c2c, Begin of try block
00000001000b5c3c         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5c40         adrp      x8, #0x100c6c000                            ; &@selector(initWithCountries:selectedCountry:appEventsLogger:loginType:)
00000001000b5c44         ldr        x0, [x8, #0xeb0]                            ; objc_cls_ref_NSBundle,_OBJC_CLASS_$_NSBundle
00000001000b5c48         adrp      x8, #0x100c5a000
00000001000b5c4c         ldr        x1, [x8, #0x698]                            ; "mainBundle",@selector(mainBundle)
00000001000b5c50         bl        imp___stubs__objc_msgSend                   ; objc_msgSend, End of try block started at 0x1000b5c38, Begin of try block (catch block at 0x1000b5d78)
00000001000b5c54         mov        x29, x29                                    ; End of try block started at 0x1000b5c50, Begin of try block
00000001000b5c58         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5c5c         mov        x20, x0
00000001000b5c60         adrp      x8, #0x100c5a000
00000001000b5c64         ldr        x1, [x8, #0x6a0]                            ; "localizedStringForKey:value:table:",@selector(localizedStringForKey:value:table:)
00000001000b5c68         adrp      x2, #0x100b74000                            ; End of try block started at 0x1000b5c54, Begin of try block (catch block at 0x1000b5d78), 0x100b745a8@PAGE
00000001000b5c6c         add        x2, x2, #0x5a8                              ; 0x100b745a8@PAGEOFF, @"lab_auth_person_street"
00000001000b5c70         adrp      x3, #0x100b6b000                            ; 0x100b6b4c8@PAGE
00000001000b5c74         add        x3, x3, #0x4c8                              ; 0x100b6b4c8@PAGEOFF, @""
00000001000b5c78         movz      x4, #0x0                                    ; argument "instance" for method imp___stubs__objc_msgSend
00000001000b5c7c         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5c80         mov        x29, x29                                    ; End of try block started at 0x1000b5c68, Begin of try block
00000001000b5c84         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5c88         mov        x21, x0
00000001000b5c8c         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5c90         ldr        x1, [x8, #0x720]                            ; "titleLab",@selector(titleLab)
00000001000b5c94         mov        x0, x19                                     ; End of try block started at 0x1000b5c80, Begin of try block (catch block at 0x1000b5d78)
00000001000b5c98         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5c9c         mov        x29, x29                                    ; End of try block started at 0x1000b5c94, Begin of try block
00000001000b5ca0         bl        imp___stubs__objc_retainAutoreleasedReturnValue ; objc_retainAutoreleasedReturnValue
00000001000b5ca4         mov        x22, x0
00000001000b5ca8         adrp      x8, #0x100c5a000
00000001000b5cac         ldr        x1, [x8, #0xa50]                            ; "setText:",@selector(setText:)
00000001000b5cb0         mov        x2, x21                                     ; End of try block started at 0x1000b5c9c, Begin of try block (catch block at 0x1000b5d78)
00000001000b5cb4         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5cb8         mov        x0, x22                                     ; End of try block started at 0x1000b5cb0, Begin of try block
00000001000b5cbc         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5cc0         mov        x0, x21
00000001000b5cc4         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5cc8         mov        x0, x20
00000001000b5ccc         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5cd0         mov        x8, sp
00000001000b5cd4         add        x20, x8, #0x20
00000001000b5cd8         adrp      x8, #_AVAudioSessionCategoryPlayback_100b28000 ; 0x100b28488@PAGE
00000001000b5cdc         ldr        x8, [x8, #0x488]                            ; 0x100b28488@PAGEOFF, __NSConcreteStackBlock_100b28488,__NSConcreteStackBlock
00000001000b5ce0         str        x8, [sp, #0x60 + var_60]
00000001000b5ce4         adrp      x8, #0x100a1f000
00000001000b5ce8         ldr        d0, [x8, #0x1c8]                            ; double_value_1_60807E_minus_314
00000001000b5cec         str        d0, [sp, #0x60 + var_58]
00000001000b5cf0         adr        x8, #0x1000b5d90
00000001000b5cf4         nop
00000001000b5cf8         str        x8, [sp, #0x60 + var_50]
00000001000b5cfc         adrp      x8, #0x100b30000                            ; 0x100b30e00@PAGE
00000001000b5d00         add        x8, x8, #0xe00                              ; 0x100b30e00@PAGEOFF, 0x100b30e00
00000001000b5d04         str        x8, [sp, #0x60 + var_48]
00000001000b5d08         add        x1, sp, #0x28
00000001000b5d0c         mov        x0, x20
00000001000b5d10         bl        imp___stubs__objc_copyWeak                  ; objc_copyWeak
00000001000b5d14         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5d18         ldr        x1, [x8, #0xa90]                            ; "selectItem:",@selector(selectItem:)
00000001000b5d1c         mov        x2, sp                                      ; End of try block started at 0x1000b5cb8, Begin of try block (catch block at 0x1000b5d68)
00000001000b5d20         mov        x0, x19
00000001000b5d24         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5d28         adrp      x8, #0x100c5b000                            ; &@selector(getLoanUseSignal)
00000001000b5d2c         ldr        x1, [x8, #0xa98]                            ; "show",@selector(show)
00000001000b5d30         mov        x0, x19
00000001000b5d34         bl        imp___stubs__objc_msgSend                   ; objc_msgSend
00000001000b5d38         mov        x0, x20                                     ; End of try block started at 0x1000b5d1c, Begin of try block
00000001000b5d3c         bl        imp___stubs__objc_destroyWeak               ; objc_destroyWeak
00000001000b5d40         mov        x0, x19
00000001000b5d44         bl        imp___stubs__objc_release                   ; objc_release
00000001000b5d48         add        x0, sp, #0x28                               ; argument "instance" for method imp___stubs__objc_destroyWeak
00000001000b5d4c         bl        imp___stubs__objc_destroyWeak               ; objc_destroyWeak
00000001000b5d50         ldp        x29, x30, [sp, #0x60]
00000001000b5d54         ldp        x20, x19, [sp, #0x50]
00000001000b5d58         ldp        x22, x21, [sp, #0x40]
00000001000b5d5c         ldp        x24, x23, [sp, #0x30]
00000001000b5d60         add        sp, sp, #0x70
00000001000b5d64         ret
                        ; endp
```