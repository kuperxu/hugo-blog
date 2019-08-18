---
title: "iOS图片渲染"
date: 2019-08-18T15:03:19+08:00
draft: true
---

# 渲染图片到屏幕上
　 每一个像素点均由三个颜色组件构成:红，蓝，绿外加一个透明度。在每个苹果产品上都有上百万个像素点需要绘制，并且需要一个稳定的FPS支撑页面的流畅度，这是一个很庞大的工作量。这是怎样一个流程？
　  iOS设备给用户视觉反馈其实都是通过QuartzCore框架来进行的，说白了，所有用户最终看到的显示界面都是图层合成的结果，而图层即是QuartzCore中的CALayer。
　通常我们所说的视图即UIView，并不是直接显示在屏幕上，而是在创建视图对象的时候视图对象会自动创建一个层，而视图对象把要显示的东西绘制在层上，待到需要显示时硬件将所有的层拷贝，然后按Z轴的高低合成最终的合成效果。

![软件对图形处理的流程](http://upload-images.jianshu.io/upload_images/1767147-c057cb90b786d4ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
对于这块详细的介绍请看：[绘制像素到屏幕上](http://objccn.io/issue-3-1/)
>GPU在相识像素方面起核心作用（在浮点运算方面做的很好）。它连接到CPU，在两者间有OpenGL，Core Animation 和 Core Graphics来做数据传输。上图的流程确保了图形的绘制。其次，在透明和不透明方面，当源纹理是完全不透明的情况下R = S + D * ( 1 – alpha )这个绘制公式，就不需要合成像素值，大大提高了性能通过CALayer的opaque来设置。ps:Quartz是iPhone OS的窗口服务器和描画技术的一般叫法。Core Graphics框架是Quartz的核心，也是内容描画的基本接口。Core Graphics就是调用drawRect()方法绘制上下文时候的一系列函数

##离屏渲染
这个知识点需要知道**当前屏幕渲染**的概念，当前屏幕渲染是指GPU在的渲染操作是在当前的屏幕缓冲区中进行渲染的。
**离屏渲染**相比当前屏幕渲染，所有不在当前屏幕缓冲区进行渲染的过程都是离屏渲染，即GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。这其中有一种特殊的渲染方式就是CPU渲染。这种情况发生在当我们对UIView的DrawRect方法进行重写的情况下并在代码中用到了Core Graphics技术进行了操作，这就是**CPU渲染**。整个CPU在App内同步完成，渲染得到的bitmap最后在提交给GPU进行显示。但是由于所有的Core Graphics都是线程安全的所以可以异步完成CPU渲染。
####性能
1.由于离屏渲染需要多次的上下文切换：先从当前屏幕切换到离屏进行渲染操作；渲染结束后，切换回当前屏幕将渲染完成之后的结果放到屏幕上。**上下文切换的代价相当大！！！**
2.离屏渲染需要新建一个缓冲区！

设置了下面的CALayer属性都会触发离屏绘制：
shouldRasterize（光栅化）
masks（遮罩）
shadows（阴影）
edge antialiasing（抗锯齿）
group opacity（不透明） 
>看到了这个masks很容易就让我想到了我们经常用的设置圆角的makeToMasks属性。嘿嘿,性能低下吧，那接下来就来高效的设置一个圆角图形。

##高效设置圆角图形
在UIImage中添加一个类别
```

-(UIImage *)jg_drawRadius:(CGFloat)radius size:(CGSize) sizetoFit{
    CGRect rect = CGRectMake(0, 0, sizetoFit.width, sizetoFit.height);//图形大小
    UIGraphicsBeginImageContextWithOptions(rect.size,false,[UIScreen mainScreen].scale);//绘制图形按尺寸，透明，比例
    
    CGContextAddPath(UIGraphicsGetCurrentContext(),[UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:UIRectCornerAllCorners cornerRadii:CGSizeMake(radius, radius)].CGPath);//添加路径
    
    CGContextClip(UIGraphicsGetCurrentContext());//裁剪内容
    
    [self drawInRect:rect];
    
    CGContextDrawPath(UIGraphicsGetCurrentContext(), kCGPathFillStroke);
    UIImage *output = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return output;
}
```
>针对于这种方法就是一定要注意这个图片的背景色不能随意设置因为在这种方法中我们刻意的避免的masks的使用，就是避免了离屏渲染，所以如果有背景色还是会影响视觉效果！core animation也不会变黄哦！

##FPS
**查看自己应用程序是否卡顿--FPS**
1.自带工具Profile的Core Animation。查看FPS
2.通过CADisplayLink

>声明：frameInterval的固定值为1，表示是一秒钟刷新60帧。duration是一帧维持的时间。CADisplayLink相比NSTimer的区别是，前者调用方法时间一定，而且相当精准。后者调用会受到runtime的繁忙程度影响。

###CADisplayLink
首先它是一个定时器，需要我们手动加入到runloop。本质上和NSTimer是一样的。但是他不同的是每次频率刷新的时候会调用方法。the selector on the target is called when the screen’s contents need to be updated.调用方法target中可以使用timestamp（时间戳）来计算FPS。可以通过时间戳的插值这样计算`1/（timestamp1-timestamp2）`计算但是由于timestamp1-timestamp2差值很小不易计算一般通过count多计算几次这样计算`count/（timestamp1-timestamp(n)）`--相当精准。
可以参考一个很好的源码:[Github](https://github.com/jvjishou/FHHFPSIndicator)