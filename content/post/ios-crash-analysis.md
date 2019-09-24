---
title: "iOS Crash 分析"
date: 2019-09-04T16:01:23+08:00
lastmod: 2018-03-05T16:01:23+08:00
draft: false
tags: ["tag-6"]
categories: ["docs", "iOS"]

menu:
  main:
    parent: "docs"
    weight: 2
---


## What Crash Is
APP被强制中断的行为视为Crash。数据上报中有完整的堆栈时我们可根据代码来分析出Crash原因，进行修正。当堆栈中出现了系统堆栈或是堆栈并不完整，我们应该如何分析，以下对此情况进行分析讨论。

## 寄存器

处理系统堆栈或是堆栈不完整的上报时，我们需要关注寄存器的信息。寄存器讨论针对ARM64架构下。iOS下大抵包含两类寄存器。

1.``r0 - r30`` 31个通用整形寄存器，寄存器大小为64位。没有名为X31或W31的寄存器。 一些指令被编码了，以使数字31代表零寄存器ZR（WZR / XZR）。 还有一组受限制的指令，其中一个或多个参数被编码，以使数字31代表堆栈指针（SP）。
2.``V0 - V31`` 32个向量寄存器，也就是是浮点型寄存器，寄存器大小为128位。

其中关于一些寄存器代表的信息中：x29就是FP, x30就是LR，这里会有人指出x31代表着SP寄存器上面也有解释，这只是个代号实际上并无x31寄存器。

- SP（Stack Pointer）栈顶寄存器，指向栈顶也就是低地址。在指令编码中，使用 SP/WSP来进行对SP寄存器的访问。
- FP（Frame pointer）栈底寄存器，指向栈底也就是高地址。
- PC（Program Counter）程序计数器，存的是当前执行的指令的地址。在arm64中，软件是不能改写PC寄存器的。
- LR (link register) 链接寄存器，当调用函数时，返回地址即PC的值被保存到LR中。
- SPSR 状态寄存器，用于存放程序运行中一些状态标识。不同于编程语言里面的if else.在汇编中就需要根据状态寄存器中的一些状态来控制分支的执行。状态寄存器又分为 The Current Program Status Register (CPSR) 和 The Saved Program Status Registers (SPSRs)。 一般都是使用CPSR， 当发生异常时， CPSR会存入SPSR。当异常恢复，再拷贝回CPSR。
- CPSR (current program status register) 程序状态寄存器。cpsr在用户级编程时用于存储条件码；CPSR包含条件码标志，中断禁止位，当前处理器模式以及其他状态和控制信息。
- ESR/FAR 异常寄存器

***一般来讲ARM64的机型 x0 – x7 分别会存放方法的前 8 个参数。如果是iOS的方法调用，x0、x1会分别保留隐藏参数self & _cmd.如果参数个数超过了8个，多余的参数会存在栈上，新方法会通过栈来读取。***

***方法的返回值一般都在 x0 上，如果方法返回值是一个较大的数据结构时，结果会存在 x8 执行的地址上。***

>关于详细的iOS函数调用时各寄存器的作用可见文章[iOS函数调用](https://blog.cnbluebox.com/blog/2017/07/24/arm64-start/)


有了这些知识我们就可以通过crash时寄存器的值来获取更多的有用信息，crash的一次寄存器信息如下。

```
Thread 0 crashed with ARM 64 Thread State:
     x0:  0x000000013555e980    x1: 0x000000024582c242    x2: 0x000000012e270df0     x3: 0x0000000000000040
     x4:  0x000000012e270e40    x5: 000000000000000000    x6: 0x0000000000000062     x7: 0x00000000000001a0
     x8:  0x0000000000000230    x9: 0x0000000114cc7f40   x10: 0x000000011107ca00    x11: 0x00000078000000ff
    x12: 0x0000000354856a04    x13: 0x200000001355ba12   x14: 0x0000000000000006    x15: 0x0000000000000001
    x16: 0x000000001355ba10    x17: 0x001a750217da5dc4   x18: 000000000000000000    x19: 0x0000000134a5d4a0
    x20: 0x000000024582c242    x21: 0x000000010aac7e5f   x22: 0x0000000106526568    x23: 0x0000000107617000
    x24: 0x00000002521cf000    x25: 0x00000002521cf000   x26: 0x0000000100effb68    x27: 0x0000000248b3bc00
    x28: 0x0000000000000001     fp: 0x000000016f008e60    lr: 0x00000001054bcc7c    
     sp: 0x000000016f008e30     pc: 0x00000002170cb6b0  cpsr: 0x20000000
    esr: 0x92000006            far: 0x000000001355ba20

x-detect:   setBackgroundColor: 

Binary Images:
0x100fe4000 - 0x1061abfff  MainProject arm64 <1f27f974e5bc3d709885c2a614886aba> /private/var/containers/Bundle/Application/C1317DCE-EF83-4FB6-A722-B483B3951961/QQ.app/Frameworks/MainProject.framework/MainProject
0x109188000 - 0x10ab97fff  TlibDy arm64 <c80a1bc104f6361e9d46b12ef6a44e06> /private/var/containers/Bundle/Application/C1317DCE-EF83-4FB6-A722-B483B3951961/QQ.app/Frameworks/TlibDy.framework/TlibDy
0x217ab6000 - 0x217ac0fff  libsystem_pthread.dylib arm64e <829ee40698373e81aab436c9f70b6726> /usr/lib/system/libsystem_pthread.dylib
0x217a1d000 - 0x217a48fff  libsystem_kernel.dylib arm64e <f2a2738e22e8381ebef30a0556212114> /usr/lib/system/libsystem_kernel.dylib
0x2178f4000 - 0x21791efff  libdyld.dylib arm64e <5666efd3f0dd3b1cae0b0fac7e92121c> /usr/lib/system/libdyld.dylib
0x217941000 - 0x2179c0fff  libsystem_c.dylib arm64e <75ac4c9d5bee3060b6da8cfa1bddb2a2> /usr/lib/system/libsystem_c.dylib
0x2170ae000 - 0x217835fff  libobjc.A.dylib arm64e <e8055992eb2530e0b15a82af9bd334e4> /usr/lib/libobjc.A.dylib
0x217041000 - 0x217099fff  libc++.1.dylib arm64e <d4063e21f493336286465788ebae505b> /usr/lib/libc++.1.dylib
0x217d91000 - 0x2180f4fff  CoreFoundation arm64e <23157e07bf733823ba6d1816915ffece> /System/Library/Frameworks/CoreFoundation.framework/CoreFoundation
0x218812000 - 0x218b0cfff  Foundation arm64e <fc334924845d3e44a4aab11d9af6cdef> /System/Library/Frameworks/Foundation.framework/Foundation
0x21845a000 - 0x218811fff  CFNetwork arm64e <7bac5947136d32178a2c8ba7bb9ed387> /System/Library/Frameworks/CFNetwork.framework/CFNetwork
0x21a06f000 - 0x21a081fff  GraphicsServices arm64e <76e795a126ce361ebb0ce1a8b674bddb> /System/Library/PrivateFrameworks/GraphicsServices.framework/GraphicsServices
0x2449de000 - 0x245b1dfff  UIKitCore arm64e <19fd9883fb333736aa548b4f3b4a1958> /System/Library/PrivateFrameworks/UIKitCore.framework/UIKitCore
0x21f15f000 - 0x21fe97fff  JavaScriptCore arm64e <8f2ae7aacd1236018362df5ae43e0d5e> /System/Library/Frameworks/JavaScriptCore.framework/JavaScriptCore
0x220957000 - 0x2223b1fff  WebCore arm64e <1763a71221433e1a8220577707d6f002> /System/Library/PrivateFrameworks/WebCore.framework/WebCore
0x21ddf8000 - 0x21deedfff  AVFAudio arm64e <f68c9dba3bad38dfbf4a959b06253505> /System/Library/Frameworks/AVFoundation.framework/Frameworks/AVFAudio.framework/AVFAudio
0x21d86b000 - 0x21da8efff  CoreMotion arm64e <457d54813e293412939c0799ab6e0658> /System/Library/Frameworks/CoreMotion.framework/CoreMotion
0x22c07a000 - 0x22c4c4fff  SceneKit arm64e <a339b467eda639b6818b5a517aaf0aa9> /System/Library/Frameworks/SceneKit.framework/SceneKit
0x21a081fff  GraphicsServices arm64e <76e795a126ce361ebb0ce1a8b674bddb> /System/Library/PrivateFrameworks/GraphicsServices.framework/GraphicsServices
0x2449de000 - 0x245b1dfff  UIKitCore arm64e <19fd9883fb333736aa548b4f3b4a1958> /System/Library/PrivateFrameworks/UIKitCore.framework/UIKitCore
0x21f15f000 - 0x21fe97fff  JavaScriptCore arm64e <8f2ae7aacd1236018362df5ae43e0d5e> /System/Library/Frameworks/JavaScriptCore.framework/JavaScriptCore
0x220957000 - 0x2223b1fff  WebCore arm64e <1763a71221433e1a8220577707d6f002> /System/Library/PrivateFrameworks/WebCore.framework/WebCore
0x21ddf8000 - 0x21deedfff  AVFAudio arm64e <f68c9dba3bad38dfbf4a959b06253505> /System/Library/Frameworks/AVFoundation.framework/Frameworks/AVFAudio.framework/AVFAudio
0x21d86b000 - 0x21da8efff  CoreMotion arm64e <457d54813e293412939c0799ab6e0658> /System/Library/Frameworks/CoreMo
```

## DSYM

知道了寄存器的值还不够，寄存器中如果存储基础类型可以直接读取，但寄存器一般存储的都是对象的地址，或是方法的地址。如何符号化。那么就要用到 DSYM（debug symbol） & atos。这部分也可自行google使用。使用命令大致贴一下：

```
Binary Images:
0x100fe4000 - 0x1061abfff  MainProject arm64 <1f27f974e5bc3d709885c2a614886aba> /private/var/containers/Bundle/Application/C1317DCE-EF83-4FB6-A722-B483B3951961/QQ.app/Frameworks/MainProject.framework/MainProject  //Crash APP 版本的UUID

dwarfdump --uuid Your.app.dSYM //DSYM的UUID
两个对比确保一致性。

atos -o yourAppName.app.dSYM/Contents/Resources/DWARF/yourAppName -arch arm64/armv7 -l <load-address> <address>
//翻译堆栈命令
```
上述只能符号化非系统的代码段地址，系统库地址的符号化要通过对应系统版本和架构的符号表得出。

## 尾调优化

crash的堆栈有时并不像实际代码调用情况，这其中一个原因是尾调优化（其他原因暂未找到请各位赐教）。尾调优化内容可以参考：
[尾调优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)

尾调用优化的本质就是栈帧的复用。
函数调用的过程：函数调用会在内存中申请一块“栈帧”，保存调用的地址和内部变量等信息。如果函数A内部调用函数B，那么在函数A的栈帧上就会加上一个函数B的栈帧。
如果函数B再调用了函数C，那么函数A的栈帧上就会有序加上函数B和函数C的栈帧。如果C运行结束了，返回到函数B，C的栈帧才会消失。

尾调用优化的条件有三点：

- 尾调用函数不需要访问当前栈帧中的变量。（变量是形参可以，变量是实参不行）
- 尾调用返回后，函数没有语句需要执行。（最后一步仅仅只能执行一个函数）
- 尾调用结果就是函数的返回值。（不能有别的“附加品”，最后一步仅仅只能是执行一个函数）


## 实战分析
### 1.现网出现crash堆栈如下
```
0 libobjc.A.dylib 0x00000001cee7f6b0 objc_msgSend + 16
1 Foundation 0x00000001d06db110 ___57-[NSNotificationCenter addObserver:selector:name:object:]_block_invoke_2 + 16
2 CoreFoundation 0x00000001cfbd11d0 ___CFNOTIFICATIONCENTER_IS_CALLING_OUT_TO_AN_OBSERVER__ + 16
3 CoreFoundation 0x00000001cfbd1190 ____CFXRegistrationPost_block_invoke + 56
4 CoreFoundation 0x00000001cfbd0640 __CFXRegistrationPost + 420
5 CoreFoundation 0x00000001cfbd02c0 ____CFXNotificationPost_block_invoke + 88
6 CoreFoundation 0x00000001cfb46e70 -[_CFXNotificationRegistrar find:object:observer:enumerator:] + 1496
7 CoreFoundation 0x00000001cfbcfd70 _CFXNotificationPost + 712
8 Foundation 0x00000001d05cc6b0 -[NSNotificationCenter postNotificationName:object:userInfo:] + 68
9 MainProject 0x0000000107011eb0 -[QQThemeManager reloadCustUIAppearance:] (QQThemeManager.mm:1060)
```
主题切换时发送通知到业务，业务方本应释放的对象接到了通知，也就是野指针。主题通知全业务都会接收到一搜索大概有30处左右的通知，一个个排查不现实，需要定位到具体调用方。

### 2.寄存器分析
1.结合上述寄存器所代表的信息，分析出x1对应的crash方法，经过系统符号化得到方法 setBackgroundColor: 与 x-dectect 检测的方法一致，说明堆栈和寄存器信息正确。

2.LR代码栈帧调用的返回地址，也就是可以定位到在哪个方法里面调用了 setBackgroundColor 方法，通过对应APP版本的DSYM分析出是登录业务的 handleThemeChange: 方法。

3.分析业务代码，文件使用MRC内存管理，嗯~真香。分析几个全局变量很简单就发现了，对象的使用不当之处。初始化变量使用的autoRelease方法，对象并没有真正被self持有，一旦被使用完这个对象就彻底被释放了，在访问也会马上Crash。


## 字节小感

目前bit和byte的比较
bit:

计算机中的最小存储单元
存储内容总是0或1
所有二进制状态的实体都可以使用1bit表示
8bits组成1byte
不能够单独寻址
byte：

1byte包含8bits
可以存储所有ASCII所有字符（这是它包含8bits的初衷）
十进制整数范围[-128,127]或[0, 255]
最小的可寻址存储单元。

iOS最小内存分配16B。