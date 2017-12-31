---
layout:     post
title:      "颜色渐变<gradient>标签之坑爹的 @android:color/transparent"
subtitle:   " \"Belongs to Android\""
date:       2017-12-21 11:30:00
author:     "小五"
header-img: "img/home_bg2.jpeg"
tags:
    - Android
---

## 颜色渐变<gradient>标签之坑爹的 @android:color/transparent

>**先来切入一个场景：**
>
>[小五][]童鞋想要一个直线的渐变，从左到右，颜色从绿色逐渐变成透明的，按照这个想法，[小五][]童鞋写下了如下代码:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <!--gradient 中 android:angle数据类型为Integer，代表渐变颜色的角度。
    0 表示从左到右, 90从下到上 . 逆时针方向依此类推即可获得各种角度变化 但其取值必须        
    是45的整数倍.且该属性只有在android:type="linear"情况下起作用，默认的type为linear。
    android:angle默认取值为0. 表示从左到右 --> 
    <gradient 
        android:type="linear"
        android:startColor="#27ae60"
        android:endColor="@android:color/transparent"/>
</shape>
```
**然后，请提前拖住下巴，我们一起瞧瞧效果图：**
![绿色到透明的效果图](http://upload-images.jianshu.io/upload_images/2378059-2071253b97257ee3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **怎么肥事！太Ugly了吧，中间夹杂的黑灰色东东从哪冒出来的？** 
---

**哈哈，惊不惊喜，意不意外？** 其实道理很简单，我们只需要用照妖镜看看`@android:color/transparent`的真面目就明白了。且看下图`@android:color/transparent `之素面朝天照。
![素颜照](http://upload-images.jianshu.io/upload_images/2378059-06c4930bba7804c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>**真相：**`@android:color/transparent `的十六进制颜色表示是**`#00000000`**，这货不就是一个**全透明的黑色**么！

## 是哒是哒 :smile: ~ 哈哈 这就是那坨黑灰色东东的来源所在！
---
大家应该都知道，我们的十六进制颜色数值由ARGB[^注脚]四个部分组成.所以我们[小五][]童鞋写的从**绿色**到**透明色**渐变的代码最终执行过程是这样的：
![小五童鞋代码中startColor到endColor对应ARGB数值的变化过程](http://upload-images.jianshu.io/upload_images/2378059-c3aa2e28f8e995ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**上图颜色数值的变化：A:`FF->00` R:`27->00` G:`AE->00` B:`60->00`**

现在，看着这个颜色渐变过程演示图，大家是不是立马豁然开朗，茅塞顿开呀 哈哈
[小五][]童鞋的代码之所以有**黑灰色杂质**，就是因为坑爹的**`@android:color/transparent `**虽然是全透明的，但却是一个**黑色的全透明**，而代码中的绿色`#27ae60`[^注释]到**`#00000000`**的渐变过程中，其对应的RGB也在变化，绿色`#27ae60`[^注释]的Alpha值是到颜色渐变的最后才变为0的， 也就是中间的颜色渐变过程中Alpha并不等于0，因此颜色渐变的各个阶段所对应呈现出来的颜色效果最终都会**参杂着RGB数值变化带来的颜色污染**，就像这里的**黑灰色杂质**。

## 所以，[小五][]童鞋该怎么做，才能实现想要的效果？
>扫地僧：**控制色域变化范围**

也就是只让ARGB中的A变化，其余的RGB保持不变！一言不合上代码：
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <gradient 
        android:type="linear"
        android:startColor="#27ae60"
        android:endColor="#0027ae60"/>
</shape>
```
修改后的效果图：**终于绿彻底了 ！**

![小五想要的效果图](http://upload-images.jianshu.io/upload_images/2378059-dc99e7703ba547e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### OL ! 就酱紫吧，希望对大家有所帮助，如果喜欢，不要小气，给个呗！

[^注脚]: **ARGB**分别代表***Alpha*** 和 ***Red*** 、***Green*** 、***Blue*** 四种色彩空间
[^注释]: `#27ae60`中缺失的Alpha值会自动用FF补全，也就是`#FF27ae60`

[小五]: http://www.jianshu.com/u/b9cbfe0a7f35  "就是我啦 哈哈"