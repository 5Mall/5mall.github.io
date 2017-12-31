---
layout:     post
title:      " Android Studio之神奇魔法Live Templates，让编码从此飞起来~"
subtitle:   " \"Hello World, Hello Blog\""
date:       2017-01-13 18:22:00
author:     "小五"
header-img: "img/post-bg-2015.jpg"
tags:
    - Android
---
## Android Studio之神奇魔法Live Templates，让编码从此飞起来~

如果你是一位安卓程序员，你一定不会对Toast感到陌生，你有没有在编写Toast千百次之后，感觉到了一丝丝厌倦？你有没有即使敲了千百遍，依然偶尔会犯些低级错误，比如将Toast写出这样：

> Toast.*makeText*(MainActivity.this, "神啊，赐我一个妹子吧！");



可能机智的你早已看穿一切！不屑一顾的说，直接写个工具类封装一下不就OK了，于是你写了下面这些代码（图1）：

![图1](https://upload-images.jianshu.io/upload_images/2378059-356b840572d177fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)



没错，这的确可以让你避免犯下上面所说的错误，但是你依然无法逃脱调用AppUtils.showTxt(arg1,arg2) **千百遍 **之后的厌倦感。而且日常编码中重复性的代码又不仅仅只有Toast这一条，你总不能每次都这样封装吧。                                                       

### 所以勒？嗯，没错，动态模板（Live Templates）!!!  

来看看通过使用动态模板之后，每次编写Toast的效果（图2）：

![图2](https://upload-images.jianshu.io/upload_images/2378059-6296e806099fbbb8.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

>只需要输入**Toast+Tab键 **！也许你会不以为然？但是如果你需要输入的重复性代码是几行甚至几百行的代码块呢？

**所以，还在等什麽哦，赶紧用起来吧骚年们！让那些高频的重复性编码工作从此狗带吧！！！**

这些是**Android Studio**内置的一些动态模板，当然只是冰山一角，详细可以查看IDE的设置界面找到全部的内置模板

![图3：只是冰山一角哦](https://upload-images.jianshu.io/upload_images/2378059-c48fa5d7ea6a2c37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

如果你在自己的**Android Studio**中找到了动态模板的设置界面，大致应该是这样的（图4），我想不管是**window**下还是**Mac**下该页面应该都大同小异。

![图4：可以看到还支持AndroidXml 文件的动态模板哦](https://upload-images.jianshu.io/upload_images/2378059-a599762d2bfd284b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

>没错，看的仔细的童鞋可能发现了，动态模板可以**自定义添加和修改！！！碉堡了有没有！**

并且支持所有动态模板的默认扩展按键的自定义，默认是`Tab`,还支持`Space`和`Enter`键 ，总共三个可选。然后针对每一条模板，可以设置独立的扩展按键（如图5）。



![图5](https://upload-images.jianshu.io/upload_images/2378059-743a5e031dfcaa53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

**好啦，大致就这么多吧，希望对大家有用^_^，Ps:纯手写，不足之处，还望指正** 



> 最终幻想：总幻想有一天粗线一位神秘的热衷打赏的土豪，哈哈~

