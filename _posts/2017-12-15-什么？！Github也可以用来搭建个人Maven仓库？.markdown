---
layout:     post
title:      "什么？！Github也可以用来搭建个人Maven仓库？"
subtitle:   " \"Hello World, Hello Blog\""
date:       2017-12-15 17:00:00
author:     "小五"
header-img: "img/home_bg.jpg"
tags:
    - 教程
---

我记得刚开始入门做程序员那会儿，很喜欢自己琢磨，所以项目中遇到了需要实现的功能，总是会先尝试着自己编码来实现_（矮油，小五童鞋很牛逼啊，刚入门就是高起点呐，简直屌得不要不要的嘛！）_， 哈哈，说来惭愧，其实之所以这样做 ，恰恰是因为自己当时太嫩，还不太会利用GitHub找轮子， 也因为如此，自己经历了一段黑暗不堪回首滴重复造车轮子滴苦逼岁月, __哭泣ing__ , 虽然话是这么说，但现在回想起来，当时自己也算是苦中作乐确有收获了。

​	不好意思，好像说了一堆废话，哈哈，见谅见谅，我们现在就回归正题。

​        之前在做公司的一个仓库项目管理的APP的时候，需要使用数据库在手机端保存数据，所以我就在项目中使用了大名鼎鼎的[greenDAO3][]依赖来完成这一块的功能逻辑开发，然后问题出现了，[greenDAO3][]本身的异步操作功能实现是通过引用旧版本的[Rxjava][]来完成的，和当时这个项目中已经在使用的新版本[Rxjava2][]就一下子不和谐了，虽然他们可以同时存在，不影响项目的构建，但无疑俩个功能高度雷同的俩代库会让项目变得臃肿，为此，我想着fork一下[greenDAO][]的源码，然后将里面的[Rxjava][]依赖更新到[Rxjava2][], 但在其项目的[issue][1]中发现，已经有一个叫[AndroidClasses][]人这样做过了，于是自己就使用[JitPack][]将这位童鞋的代码生成了一个maven依赖直接引入解决了这个问题。

​	![使用JitPack 生成的依赖代码][img01]
 ***PS:***  _此处建议比较好的做法是先将AndroidClasses的项目代码fork到自己的GitHub然后使用JitPack来生成对应的maven依赖，这样可以保证项目依赖的安全性，防止这位童鞋哪天失恋心情不好，直接删库带来一些不必要的麻烦_

---

 	什么，说了这么多废话，原来就一关键字[JitPack][]完事啊，哈哈 ，的确，大多时候，我们只要将项目需要的第三方库在[Github][]上的代码fork到自己的github库 然后将地址提交到[JitPack][]即可获取对应的Maven依赖地址。但一般使用像[greenDAO][]，[Fresco][]等牛逼的大型开源项目我们常常不会使用到[JitPack][]，因为他们一般都会有人维护所以还是很稳定的。就我个人来讲，通常会使用到[JitPack][]来获取maven依赖的对象大都是自己发布在[Github][]的项目以及一些存在不稳定因素的个人开源项目还有那些需要做一些定制化的修改优化和bug修复的第三方依赖库。

------

##**OL，完事了？NO！难道你就不想自己实现一个[JitPack][]？**


​        哈哈，别紧张，我说的不是让大家自己建一个[JitPack][]网站，我的意思是我们能不能像[JitPack][]一样利用[Github][]来搭建一个属于自己的Maven仓库...因为如果万一...[JitPack][]挂掉了（呃，我咋这么坏 哈哈 ）我们也有个备份选择不是 。

​        **Okay，Let's get started ! 快上车！**

​        **1**、准备在[Github][]上创建俩仓库[abstractlistadapters][]和[mvn-repo][]。[abstractlistadapters][]是我们需要发布到Maven仓库的Android Library项目，[mvn-repo][]用来存放[abstractlistadapters][]的Maven部署文件。*(ps:这个[mvn-repo][]仓库还可以放其它项目的Maven部署文件，也就是说它可以专门用来存放各种项目对应的Maven部署文件，是一个Maven部署文件存放仓库）在本例中我会把[abstractlistadapters][]项目的maven部署文件release版本放置在__mvn-repo/Android/Lib/abstractlistadapters/__ 下，snapshot版本放置在mvn-__repo/Android/Lib/abstractlistadapters/snapshots/__ 下。这个路径很重要，如果你看到本文章结尾，你就会发现maven对应的URL与此路径讲保持一致。

​        **2**、打开AndroidStudio,新建一个Android Library项目，在项目根目录下找到**gradle.properties **文件，如果没有，请新建一个。在该文件下粘贴如下代码：
```properties
VERSION_NAME=0.0.1-SNAPSHOT
VERSION_CODE=1
GROUP=com.walkermanx.android.lib  //改成自己的

POM_DESCRIPTION="this is my first android aar mvn repo in github !"
#对应要发布项目，此处是 abstractlistadapters 的github地址
POM_URL=https://github.com/walkermanX/abstractlistadapters    
#对应要发布项目，此处是 abstractlistadapters 的github地址
POM_SCM_URL=https://github.com/walkermanX/abstractlistadapters
#abstractlistadapters 的git地址
POM_SCM_CONNECTION=scm:git@github.com:walkermanX/abstractlistadapters.git
POM_SCM_DEV_CONNECTION=scm:git@github.com:walkermanX/abstractlistadapters.git
POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo
POM_DEVELOPER_ID=walkermanX  //发布者id
POM_DEVELOPER_NAME=walkermanX <zhangwei-baba@163.com> //发布者名字

#项目的Maven本地部署路径，部署文件生成之后会存放在此，然后我们把这些文件提交到之前提前在githun上建好的mvn_repo仓库

RELEASE_REPOSITORY_URL=file:///D:/WorkSpace/AndroidStudioProject/personal_projects/mvn-repo/Android/Lib/abstractlistadapters/
SNAPSHOT_REPOSITORY_URL=file:///D:/WorkSpace/AndroidStudioProject/personal_projects/mvn-repo/Android/Lib/abstractlistadapters/snapshots/

#RELEASE_REPOSITORY_URL 和 SNAPSHOT_REPOSITORY_URL
#两个配置分别是正式版和快照版的本地路径。如果版本号后带有 -SNAPSHOT (大小写敏感  也就是-snapshot依然会发布到 RELEASE_REPOSITORY_URL 目录下 不会发布发到SNAPSHOT_REPOSITORY_URL )
#编译后会发布到 SNAPSHOT_REPOSITORY_URL 相应的目录下。
```

   **3** 、通过2中链接获取到上图代码后并按照截图说明对应修改，之后在项目对应要发布的Library Module下再次创建一个__gradle.properties__ 文件（下图B部分）并粘贴如下代码：

```properties
POM_NAME=Library-abstractlistadapters  //
POM_ARTIFACT_ID=abstractlistadapters
POM_PACKAGING=aar 
```

![代码截图说明][img02]

  **4** 、在Library Module下的build.gradle文件中调用[gradle-mvn-push][] 插件 ，如下图

```properties
apply from: 'https://raw.githubusercontent.com/walkermanX/gradle-mvn-push/master/gradle-mvn-push.gradle'
```

![代码截图说明][img03]

  **5** 、上图中脚本获取方式如下图：
![代码截图说明][img04]

  **6** 、点击打开项目底部的Terminal栏，并执行`gradlew clean build uploadArchives` 命令后会在会在`RELEASE_REPOSITORY_URL`对应的路径创建相关的文件，然后在`AS`中本地`Android Library`项目的代码提交到[Github][]上的[abstractlistadapters][]仓库，将[Github][]新建的[mvn-repo][]仓库`clone`到本地路径A，并将`RELEASE_REPOSITORY_URL`对应路径生成的文件拷贝到路径A下对应该项目的文件夹中然后执行`git`命令提交到远程[mvn-repo][]仓库即可。
  **7**、OK，到此我们的个人Maven仓库就建好了，下面我们可以按照如下步骤在实际项目中引用啦！碉堡了有木有 ^_^
* 在项目的根目录中找到 __`build.gradle`__ 文件 （*不是module的`build.gradle `文件* ），加入如下代码：

   ```groovy
   allprojects {
       repositories {
           maven {
               url 'https://raw.githubusercontent.com/walkermanx/mvn-repo/master/Android/Lib/abstractlistadapters/'
           }
       }
   }
   ```

*  在你的项目module的**`build.gradle`** 文件中引入依赖：

   ```groovy
    //建议AndroidStudio3.0以上版本 根据需要使用implementation或者api方式引入依赖
    implementation 'com.walkermanx:abstractlistadapters:0.0.1'
    //AndroidStudio3.0以下版本
    compile 'com.walkermanx:abstractlistadapters:0.0.1'
   ```


> **说明：**
> **jar：**仅仅包含class和清单文件，***没有***资源文件。
> **aar：**包含了class和清单文件以及**资源文件**。
> 因为[abstractlistadapters][]依赖目前没有引用的资源文件 所以我们没有加**@aar**后缀



**如果这篇文章对您有所帮助，希望您能大方给个心♥，您小小的鼓励对我来说是大大的动力哦\^_\^**  





[1]: https://github.com/greenrobot/greenDAO/issues/520
[2]: https://github.com/greenrobot/greenDAO	"greenDAO"

[Github]:https://github.com/  "Github"
[greenDAO]:https://github.com/greenrobot/greenDAO	"greenDAO"
[greenDAO3]:https://github.com/greenrobot/greenDAO	"greenDAO"
[Rxjava]:https://github.com/ReactiveX/RxJava "Rxjava"

[Rxjava2]:https://github.com/ReactiveX/RxJava "Rxjava"
[AndroidClasses]: https://github.com/AndroidClasses/greenDAO/tree/rxjava2-de  "greenDao with Rxjava2"
[JitPack]:https://jitpack.io/ "JitPack"
[Fresco]:https://github.com/facebook/fresco "Fresco"
[mvn-repo]:https://github.com/walkermanX/mvn-repo/tree/master/Android/Lib/abstractlistadapters  "mvn-repo"
[abstractlistadapters]:https://github.com/walkermanX/abstractlistadapters "abstractlistadapters"
[gradle-mvn-push]:https://github.com/walkermanX/gradle-mvn-push "gradle-mvn-push"
[img01]:http://upload-images.jianshu.io/upload_images/2378059-42d622a6f626b598.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700  "使用JitPack 生成的依赖代码"
[img02]:http://upload-images.jianshu.io/upload_images/2378059-114e94244985f092.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240   "代码截图说明"
[img03]:http://upload-images.jianshu.io/upload_images/2378059-1c51d49df4f63263.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240   "代码截图说明"
[img04]:http://upload-images.jianshu.io/upload_images/2378059-a7a39bc286383ff7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240  "图中脚本获取方式"



