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

