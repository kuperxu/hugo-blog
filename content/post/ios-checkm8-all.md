---
title: "iOS Device Checkm8"
date: 2019-09-29T11:42:22+08:00
draft: false
---

## checkm8

> Checkm8 iPhone上的史诗级硬件漏洞，能够让iPhone 4s到iPhone X实现永久越狱，且无法修补。[Source Code](https://github.com/axi0mX/ipwndfu)

### 关于越狱
越狱breakjail：使用户设备获得超级管理员权限，拥有对系统底层的读写权限。绕过苹果的安全监测安装非签名的APP、自定义系统。关注越狱动态可以看[reddit论坛](https://www.reddit.com/r/jailbreak/)。

[详细越狱原理介绍](https://zhuanlan.zhihu.com/p/32766546)。

**越狱是利用了两个漏洞.**
bootrom漏洞：在bootrom期间完成漏洞利用。他不能通过传统的固件更新来修补，必须由新硬件修补。由于漏洞在启动环节发生的很早，而且漏洞Payload拥有对硬件的全部读取权限。 如它可以利用AES硬件引擎的GID密码来解密IMG3文件，而IMG3文件允许解密新的iOS更新。由于他几乎在任何检查点之前，恶意代码在所有事情之前被注入，因此允许创建通道以绕过所有检查或者简单地禁用他们。

用户空间漏洞：在内核加载期间或之后完成漏洞利用，并且可以通过软件更新轻松的被Apple修补。由于这是所有的检查，它将恶意代码直接注入到内核中。这些开口不是很容易找到，一旦发现可以修补。

https://www.jianshu.com/p/c5c22f9a06e2 ..完美越狱非完美越狱

**每个人都知道的默认密码**
关于iOS最严重的秘密之一就是它的root密码“alpine”。每个人都知道，苹果也不打算改变它。使用root密码给用户访问设备的核心功能，如果落入不法之徒，这可能是灾难性的。

幸运的是，这个密码可以从一个shell应用程序中更改，但是后期的破解者经常忘记这样做，从而使他们的设备面临漏洞。

## 主角Checkm8
原理：利用bootrom漏洞进行的越狱。该漏洞是针对iPad或iPhone上启动ROM的某个向量指针。

价值：bootrom漏洞的特性：不可修复，可玩性高。从而可以得出以此设备可以刷任何版本的固件，当然包含降级，甚至可以刷安卓，还可以做远程调试。
影响：对普通消费者杀伤性几乎为零，生效条件严格：用数据线和电脑连接，设备进入DFU mode。因破解是通过启动ROM的向量指针，并非是撬开iOS系统，因此是非完美越狱，这意味着每次重新启动iPhone时，攻击媒介都会再次关闭。理论上不会被APT攻击装一个持久化的后门(有待分析)。

PS:APT即高级可持续威胁攻击,也称为定向威胁攻击，指某组织对特定对象展开的持续有效的攻击活动。 这种攻击活动具有极强的隐蔽性和针对性,通常会运用受感染的各种介质、供应链和社会工程学等多种手段实施先进的、持久的且有效的威胁和攻击。

> 思考：漏洞公布的时间点很值得考量，iPhone 11刚刚发布完成，这时发表声明3GS-iPhone X所有设备都有不可修复问题，新机XS及11是没问题的，是苹果的压迫还是说出售新机kpi的任务难以完成，当然这些各抒己见。不过从漏洞提出者的Git声明上可以看出“axi0mX觉得自己不是第一个发现这个问题的人，该漏洞很可能已经被一些其他公司利用了”。