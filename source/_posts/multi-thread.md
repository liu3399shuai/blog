---
layout: post
title: "multi-thread"
date: 2016-03-21 12:44:00 +0800
comments: true
tags:
- 线程安全
- 线程的分类
categories:
- [多线程]
- [OC]
---

线程是CPU分配和调度的基本单位，操作系统控制CPU以时间片轮流(轮转的方式)切换(执行每一个线程)形成的，由于CPU运行很

<!-- more -->

# Multi-Thread 放大理顺逻辑流、线程流

## 线程的产生方式

线程是CPU分配和调度的基本单位，操作系统控制CPU以时间片轮流(轮转的方式)切换(执行每一个线程)形成的，由于CPU运行很快(工作可以分为5个阶段：取指令、指令译码、执行指令、访存取数和结果写回)，时间间隔小到毫秒，给用户一种同时运行的假象，线程是程序设计的逻辑层概念

![](/images/thread1.png)

## 线程的组成结构(实现方式)

一个标准的线程由线程ID(面向对象上就是线程的名字)、当前指令指针(PC)、寄存器、堆栈组成

## 线程的分类

主线程(对界面UI操作的task应该在主线程)、子线程

## 线程的生命周期(运行状态及触发)

new -> runable (ready to run) -> running -> dead (finished/cancel)中间可能会有过渡态 : wait / block / sleep

![](/images/thread2.png)

```
if([thread isCanceled]){
       [thread exit]; //  The cancel method only informs the thread that it is cancelled,It's then the responsibility of the thread itself to check this and exit
}
```

## 线程安全(线程并发)

一个时间点内，某个属性、某个方法、某段代码只能有一个线程对其操作（添、删、改、查、访问），这也是单一处理原则,检测方法是否同时被多个线程执行，在方法里面打印`[NSThread currentThread]`即可
如何保证线程安全,或者说控制线程并发 1 、线程锁(也叫互斥所mutex-lock，厕所例子)  2、优化结构：错开调用时间、两个线程间进行数据传递或归为一个线程调用

线程锁 ： NSLock系列 、 @synchronized 、 pthread 、dispatch_semaphore(信号量)

## 线程的优先级

## 线程间的运行机制

serial or concurrent ( 多个线程并发执行(线程与线程之间) )
synchronous and asynchronous
异步是任务并发执行(执行到某一方法，将方法里面的task submit到子线程，方法立即返回，程序继续执行下一行)
同步是任务顺序执行(执行完某一方法并返回值后，执行下一行)
多线程是异步的实现方式

## 线程间的通信(communication)

performSelector:onThread:withObject:waitUntilDone:  only schedules the selector to run in the default run loop mode, if the run loop is in another mode ,it won`t run until the run loop switches back to the default mode, so you can get this :  performSelector:onThread:withObject:waitUntilDone:modes:

dispatch_async(dispatch_get_main_queue(),^{…}) will run the block as soon as the main run loop returns control flow back to the event loop,it doesn`t care about modes

## iOS 中线程PK ( NSObject vs NSThread vs NSOperation+Queue vs GCD )

`在Xcode里面点击进去，看官方language programing` 呵呵，这是最原始的最全面的

丰富度 NSObject -> NSThread -> NSOperation(NSOperationQueue)

NSObject里面的 performSelectorInBackground 是最原始的开启子线程方法,制造了一堆不受管理的线程，非常不建议使用

NSThread对单个线程的生命周期状态需要手动控制，并可以做到精细化管理和查看，但线程间的问题解决不了，比如线程依赖，线程间数据同步(多线程访问同一个内存地址导致的互斥同步，这种情况需要加锁)

NSOperation 同样具有NSThread的优势，而queue可以做到对多个线程的并发执行进行(线程间)控制，比如线程间同步

GCD对多个线程并发执行的控制很强(利用 once 、semaphore、group、queue、object 、监听source)，但对单个线程的生命周期状态控制的比较粗糙，几乎没有，block提交后几乎做不了什么别的，也没法查看线程的状态，也没法撤销当前线程，适合一些简单的task

![](/images/thread3.png)

## thread(线程) & process(进程) & program(程序)

进程是操作系统进行资源分配和调度的一个独立单位
进程是由程序正文段（text）、用户数据段（user segment）、系统数据段（system　segment）共同组成的一个完整的执行环境

![](/images/thread4.png)

**（1）正文段（text）：**存放被执行的机器指令。这个段是只读的（所以，在这里不能写自己能修改的代码），它允许系统中正在运行的两个或多个进程之间能够共享这一代码。例如，有几个用户都在使用文本编辑器，在内存中仅需要该程序指令的一个副本，他们全都共享这一副本。

**（2）用户数据段（user segment）：**存放进程在执行时直接进行操作的所有数据，包括进程使用的全部变量在内。显然，这里包含的信息可以被改变。虽然进程之间可以共享正文段，但是每个进程需要有它自己的专用用户数据段。例如同时编辑文本的用户，虽然运行着同样的程序__编辑器，但是每个用户都有不同的数据：正在编辑的文本。

**（3）系统数据段（system segment）：**该段有效地存放程序运行的环境(一些控制信息)。事实上，这正是程序和进程的区别所在。如前所述，程序是由一组指令和数据组成的静态事物，它们是进程最初使用的正文段和用户数据段。作为动态事物，进程是正文段、用户数据段和系统数据段的信息的交叉综合体，其中系统数据段是进程实体最重要的一部分，之所以说它有效地存放程序运行的环境，是因为这一部分存放有进程的控制信息。系统中有许多进程，操作系统要管理它们、调度它们运行，就是通过这些控制信息。Linux为每个进程建立了task_struct数据结构来容纳这些控制信息

   程序经过编译、链接后生成一个可执行文件，一个机器代码指令和数据的集合，一个静态的实体，存储在磁盘上的一个可执行映像中。程序代表期望完成某工作的计划和步骤，它还浮在纸面上等待具体实现，而具体实现是由进程来完成的，进程可以认为是运行中的程序，进行中的程序，程序的一次执行。
   程序装入内存后，在指令指针寄存器的控制下，不断的将指令取至CPU运行，同时还会产生一些额外数据(包括程序中各种指令和数据，还有一些额外数据，比如说寄存器的值、用来保存临时数据（例如传递给某个函数的参数、函数的返回地址、保存变量等）的堆栈（包括程序堆栈和系统堆栈）、被打开文件的数量及输入输出设备的状态等等。这个执行环境的动态变化表征程序的运行)，这个完整的执行环境称为进程(一个动态的实体，代表程序的执行过程，它随着程序中指令的执行而不断地变化),参考链接http://oss.org.cn/kernel-book/ch04/4.1.htm
   


