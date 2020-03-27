---
title: "iOS Swift Grammar"
date: 2019-09-24T16:05:03+08:00
draft: true
---

### swift 语法

## swift关键字

> 所有的新增关键词，都是为了提倡更好的编程风格或是更好的编程体验。这其中大部分的关键词为糖语法。

- guard let parameters = parameters else { return urlRequest } guard let表示如果 条件不成立则执行else的逻辑。这里的代码隐藏逻辑为: if(parameters) 条件不成立即parameters为空，走else逻辑。guard必须要加return、break、continue关键词，代表好的编程习惯，参考 《禅Objective-C编程艺术》。
- if let 与 guard let 逻辑相反



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

### little knowloage

clang -x objective-c -rewrite-objc -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk file.m

```
clang -x objective-c -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 -isysroot
```