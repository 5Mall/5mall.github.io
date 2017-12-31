---
layout:     post
title:      "【笔记】Django Model类下，Field字段中，choice属性有个坑！"
subtitle:   " \"Belongs to Python\""
date:       2017-12-26 20:12:00
author:     "小五"
header-img: "img/post-bg-2015.jpg"
tags:
    - Python
---

## 【笔记】Django Model类下，Field字段中，choice属性有个坑！

### 前言

---

用Python从事后端开发的同学应该都很熟悉大名鼎鼎的Django吧, Django开发中我们常会在`models.py`文件中编写我们的数据模型类，在编写模型类的时候我们常常会根据需求用到各种`Field`字段用以映射对应的数据表列。

>于是，掉坑之旅了开始

### 鄙人不才，于某日一个神志不清的下午写下了如下代码：

```Python

class ActApplicant(Model):
    status_choices = {
        (0, '未通过'),
        (1, '已通过'),
        (2, '申请中')
    }
    act = ForeignKey(Act, verbose_name='活动', on_delete=CASCADE)
    applicant = ForeignKey(Applicant, verbose_name='申请人', on_delete=CASCADE)
    apply_time = DateTimeField('申请时间')
    status = SmallIntegerField('申请状态', choices=status_choices, default=2)

    def __str__(self):
        return self.act.title

    class Meta:
        unique_together = ('act', 'applicant')
        verbose_name = verbose_name_plural = '活动申请'

```
细心的童鞋可能已经发现了代码的问题，啧啧啧，好吧，塞心仔，这篇文章可能不适合你们...
**什么！？你左看右看上看下看就是没发现有啥不对，**哈哈 ，看来我并不孤独呐，如果你是后者，那这篇文章简直就是为你而写啊！ 
### 苦逼回忆开始...

---
那天写完上面代码之后接下来的整整三个多小时里，我被一个奇怪的现象困扰，在执行数据库同步变更操作的时候，这个表竟然一直提示有变动，而通常情况下我们在执行`python manage.py makemigrations`命令之后继续重复执行该命令就会提示`No changes detected ` 
![没完没了的提示有变动](http://upload-images.jianshu.io/upload_images/2378059-e2f465f04fcd0ace.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 最终，在`“小样儿，我还治不了你了"`的精神力驱动下，顽强滴我选择了`一个字一个字`去`死抠代码`

---
咦~~ 这里的`status_choice`怎么是个花括号，记得好像应该是圆括号啊，改过来试试....**改完执行**`python manage.py makemigrations` ，嗯，还是提示有变化，再执行，`No changes detected `   **什么？！就这样好了 !!!** 

改动后的`status_choice`
```
status_choice＝（
(0,'即将开始'),
(1,'进行中'),
(2'已结束'),
）
```
改完之后运行：
![终于正常了](http://upload-images.jianshu.io/upload_images/2378059-a2c94a2f97b862f3.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### 希望这篇文章能对粗心的童鞋们有帮助，如果喜欢，也不要小气咯，给个呗！