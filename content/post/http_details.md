---
title: "Http_details"
date: 2019-10-10T17:01:57+08:00
draft: true
---

### REST

1. REST描述的是在网络中client和server的一种交互形式；REST本身不实用，实用的是如何设计 RESTful API（REST风格的网络接口）；

2. Server提供的RESTful API中，URL中只使用名词来指定资源，原则上不使用动词。“资源”是REST架构或者说整个网络处理的核心。比如：

   GET 用来获取资源，
   POST 用来新建资源（也可以用于更新资源），
   PUT 用来更新资源，
   DELETE 用来删除资源

3. 错误码(HTTP Status Code)是确定的。
4. 返回结果是做好约束的，JSON格式。

> 除去REST规则，http其实是很自由的协议，只要终端和后台协议好传输内容所有的规则都是可变的。比如post可以把参数放到URL的PATH里面进行请求。

### POST&GET

GET请求会被浏览器主动cache，而POST不会，除非手动设置。

GET请求只能进行url编码，而POST支持多种编码方式。

GET请求在URL中传送的参数是有长度限制的，而POST么有。

GET参数通过URL传递，POST放在Request body中。