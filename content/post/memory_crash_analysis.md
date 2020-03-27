---
title: "Memory_crash_analysis"
date: 2019-11-28T15:06:01+08:00
tags: ["Crash", "iOS"]
categories: ["Crash", "iOS"]
draft: true
---

## 疑难杂症

> 近期在做手Q的多彩主题适配，开发到染色能力时发现了一个奇怪的Crash，随机Crash在主线程&子线程。引发了一些思考，记录下来。

### 问题背景

- 掌握必现路径（松心了一大截）。

- 每次Crash堆栈都各不同的。90%以上Crash在主线程。

- Debug面板报错,这里可以确认是内存出现了问题，并且是内存被某个操作做了篡改。
  ```
  malloc: Incorret checksum for freed object 0x11c3dd200: probably modified after being freed.Corrupt value :0x0
  malloc:*** set a breakpoint in malloc_error_break to debug
  ```

### 解决问题过程

1.工欲善其事必先利其器，因为是必现问题，首先想到的是利用Xcode自带的调试工具来发现问题。

在依次尝试了 Guard malloc, Zombie object, Address Sanitizer之后未发现任何有用线索。

2.尝试多次crash分析堆栈，发现主线程Crash都是随机分布，而子线程稳定crash在BubbleBgInfo类。然后分析这里的代码进行问题排查。

3.通过分析来确认问题函数。发现在工程中有一处代码是对图片做主题色分析，大致实现是取image的RGB来计算平均值，根据平均值评估图片是暗色系还是亮色系。

![image-20191128155737370](/image/image-20191128155737370.png)

4.如上图，发现问题如上，此接口有一步流程是将图片数据复制到一个malloc出来的数组。而出问题的地方在于pixels数据长度相比data中的长度要远小，这是为什么呢，相差也太大了吧。按我们正常的理解，一张图被完全Decode出来应当是RGBARGBA 这样完整的排序，可怎么会出现63792这个值系统是怎么搞得？？？




### 留下疑问

若问题不是必现又要如何来排查内存问题

问题本质为内存越界写入脏数据，为什么Guard malloc工具没有检测出来。