---
layout:     post
title:      " Django Serializers高级用法之动态修改Fields参数"
subtitle:   " \"Hello World, Hello Blog\""
date:       2017-12-29 16:58:00
author:     "小五"
header-img: "img/post-bg-2015.jpg"
tags:
    - Python
---

##Django Serializers高级用法之动态修改Fields参数

###一言不合，上代码

```python
class SlidesSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Slides
        fields = ('id', 'title', 'image', 'url', 'no',)
```
使用**Django**开发的童鞋应该很面熟上面的代码吧，因为在开发中常常会编写如上的各种**Serializer**类来为我们的**Model**类数据进行**序列化**操作。通常上面这个`SlidesSerializer`类的主要作用就是将其`Meta`类中指定Model对象`models.Slides`序列化成**由其 `fields`属性声明的那些字段组成的字符串** 
```
#序列化之后的models.Slides对象
{'id': 2, 'title': '道地良品', 'image': '/daodi.png', 'url': 'www.daodikeji.com', 'no': 'DD_BJ_001'}
```
一切都挺ok，突然有一天，[小五](https://www.jianshu.com/u/b9cbfe0a7f35)童鞋觉得除了`id`、`image`和`title`字段，其它字段都不需要序列化，于是他给`models.Slides`又写了一个精简版的**Serializer**：`SlidesLiteSerializer`
```python
class SlidesSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Slides
        fields = ('id', 'title', 'image', 'url', 'no',)

#精简序列化版
class SlidesLiteSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Slides
        fields = ('id', 'title','image',)
```

##额，好像有点Low...
---
的确，怎么可以则个样子啊，虽然也能解决问题，但要是再来个需要`id`与其它四个字段分别俩俩搭配的需求，岂不是又得整一堆Serializer...  
作为一个有点强迫症的码农，我表示不能忍啊  
于是，参照[官网api文档](http://www.django-rest-framework.org/api-guide/serializers/#dynamically-modifying-fields)，我对`SlidesLiteSerializer`进行了一番改造
```python
class SlidesSerializer(serializers.ModelSerializer):
    """
     此处的`fields`字段是用来替换上面Serializer内部Meta类中指定的`fields`属性值
    """

    def __init__(self, *args, **kwargs):
        # 在super执行之前
        # 将传递的`fields`中的字段从kwargs取出并剔除，避免其传递给基类ModelSerializer
        # 注意此处`fields`中在默认`self.fields`属性中不存在的字段将无法被序列化 也就是`fields`中的字段应该      
        # 是`self.fields`的子集
        fields = kwargs.pop('fields', None)

        super(BaseModeSerializer, self).__init__(*args, **kwargs)

        if fields is not None:
            # 从默认`self.fields`属性中剔除非`fields`中指定的字段 
            allowed = set(fields)
            existing = set(self.fields.keys())
            for field_name in existing - allowed:
                self.fields.pop(field_name)

    class Meta:
        model = models.Slides
        fields = ('id', 'title', 'image', 'url', 'no',)
```
**现在，来试试：**
```
>>> print SlidesSerializer(sides)
{'id': 2, 'title': '道地良品', 'image': '/daodi.png', 'url': 'www.daodikeji.com', 'no': 'DD_BJ_001'}
>>> print SlidesSerializer(sides, fields=('id', 'title'))
{'id': 2, 'title': '道地良品'}
>>> print SlidesSerializer(sides, fields=('id', 'title','image'))
{'id': 2, 'title': '道地良品', 'image': '/daodi.png'}
```
##Nice！！！

>**Ps：**建议将构造器部分代码实现放置于基类*Serializer*，然后将Model类对应的**Serializer**实现类继承该基类，酱紫，所有继承该基类的**Serializer**均能实现**Fields参数**的动态修改

## 希望对大家有所帮助，如果喜欢，不要小气，给个♥呗！