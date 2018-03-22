---
layout:     post
title:      " 【布局优化】喂！住在xml里的fragment，我要\"replace\"掉你！"
subtitle:   " \"Belongs to Android\""
date:       2018-02-02 19:51:00
author:     "小五"
header-img: "img/home_bg2.jpeg"
tags:
    - Android
---
##  前言

大家应该都知道，Fragment的加载方式有俩种：**静态加载**和**动态加载**；
对于这俩种，不用缩！我猜大家肯定更喜欢用后者吧，毕竟咱都是**爱用动的boy**嘛   啧啧啧~好像跑偏了 

**哈哈，颜归正传！上！代！码！**

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/rootview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:tools="http://schemas.android.com/tools">
   <!--其余代码省略-->
    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"/>

</android.support.design.widget.CoordinatorLayout>
```

眼熟不？然后配合下面这段代码

```

getSupportFragmentManager().beginTransaction().replace(R.id.container,new ProcessDetailFragment()).commit();
```
就可以实现fragment的动态加载了，是不是很开心？

#### 然鹅，当我打开Android Studio 的Layout Inspector[^脚注1]的时候，我并不怎么开森

![当前fragment加载后的布局结构](http://upload-images.jianshu.io/upload_images/2378059-c2834bbc468c6920.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 为毛fragment布局不是直接替换掉container布局而是在它里面！！！
其实一般情况也没啥，只是如果是这里，**那！就！坚！决！不！能！忍！**眼神好的童鞋可能发现了，这个fragment里面是个典型的`ViewPager+Tablayout`组合，而ViewPager里面又会有一堆fragment ,所以布局结构那叫一个复杂，这种布局哪怕多一层嵌套都是会相当影响UI加载速度滴啊，所以，必须干掉它！

### But How？

聪明的你肯定想到了 哈哈 没错！静态加载fragment！于是，代码我改改改
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/rootview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:tools="http://schemas.android.com/tools">
   <!--其余代码省略-->
   <fragment
        android:id="@+id/fragView"
        android:name="com.xxx.ProcessDetailFragment"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"/>

</android.support.design.widget.CoordinatorLayout>
```

运行代码,打开视图检查

![修改后的布局结构](http://upload-images.jianshu.io/upload_images/2378059-5affb8f7f4e973e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**怎么感觉好像故事快要讲完了？ 哈哈 ，撇了一眼标题，好像又跑题了 ！**
### 当然木有跑题！ 啰嗦了这么多，其实是在为后文作铺垫哦 
其实这是在复现当时项目开发中一步步遇到问题解决问题的过程，没有上述问题的粗现，接下来要讲的东东也就不会发生了哦  所以算是个前情概要吧 就像看电影一样  嘿嘿 大家多担待 

### 不呼不唤，我始出来！
现在视图层级已经达到我们的预期了，可是！问题又粗线了，这个项目的viewPager里面是一个个fragment，其代表的是TabLayout中一个个菜单的详情页面，每个详情页面中数据会有操作，然后当其中某个fragment中数据变化之后会影响其它fragment的数据和菜单的展示，因此，在每一次数据变化的时候，我们需要刷新整个ProcessDetailFragment！

### HOW 2 ?

最简单的办法，就是用新创建的ProcessDetailFragment替换掉目前的ProcessDetailFragment，然后新的ProcessDetailFragment会重新加载更新后的数据并展示，So Easy?

于是，按照这个想法，顺便回想一下动态加载fragment的代码，fragment动态加载的时候，是需要指定容器布局id的，代码执行后新fragment的布局会加入到这个容器布局中，所以如果我们要用新的ProcessDetailFragment替换掉原有的ProcessDetailFragment其实是不可行的！

### 什么，你不信，那我们用代码来说话！

按照动态加载fragment的方式，用新的ProcessDetailFragment替换掉原来布局中的ProcessDetailFragment，注意下面代码容器id为R.id.fragView，也就是fragment在xml中声明的id
```
  public void refresh() {
          getSupportFragmentManager().beginTransaction()
         .replace(R.id.fragView, new ProcessDetailFragment())
         .commit();
}
```
然后看看新的视图结果：
![直接替换后的布局结构](http://upload-images.jianshu.io/upload_images/2378059-3ee39733b0e72758.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也许有些童鞋会说，先remove掉原来的ProcessDetailFragment在用新的fragment执行replace就好啊，但其实如果用过静态加载fragment的童鞋肯定知道，FragmentTransaction的remove针对静态加载的fragment是无效的，而且本人也在此亲测，在执行remove后布局结构依然和上图一致

### 我有个大胆的想法

既然replace会将fragment加入到指定容器id布局之下，那么我们将这个容器id设置为原ProcessDetailFragment的父容器CoordinatorLayout的id,蓝后在父容器中remove掉旧的ProcessDetailFragment布局视图不就好了？

**OL， 开干！**

新的代码：
```
//processDetailFragment 为目前布局中加载的fragment的引用
    public void refresh() {
        if (processDetailFragment != null) {
            //新的ProcessDetailFragment
            ProcessDetailFragment fragment = new ProcessDetailFragment();
            //和老fragment中的布局参数保持一致 该方法需要自定义实现
            //该方法可保证新的fragment布局和原fragment大小,行为一致
            fragment.setLayoutParams((CoordinatorLayout.LayoutParams)         
            processDetailFragment.getView().getLayoutParams());
            //将原fragment布局视图从其父容器中删除
            ((CoordinatorLayout) findViewById(R.id.rootview))
            .removeView(processDetailFragment.getView());
            //加载新的fragment
            getSupportFragmentManager().beginTransaction()
            .replace(R.id.rootview, fragment).commit();
            //用新fragment替换原fragment的引用
            processDetailFragment = fragment;
        }
    }

```

代码运行，新的视图布局结构：

![替换后的布局结构图](http://upload-images.jianshu.io/upload_images/2378059-8de886c0890ad0d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 后记

严格来讲，这算个伪替代，但，条条大路通罗马！我们最终还是实现了我们所想所需的效果，不是吗？ 也因此文章标题中的替换带了俩引号 "replace"   

>  本文可转载，转载请注明出处

#### 如果这篇文章对亲有所帮助，希望可以亲可以给我一个！这是对我最大的鼓励！多谢^^


[^脚注1]: Windows -> Tools -> Android ->Layout Inspector 即可打开布局检查，可以查看视图的布局结构