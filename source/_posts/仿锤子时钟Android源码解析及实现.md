---
layout: android
title: 仿锤子时钟Android源码解析及实现
date: 2017-05-02 00:15:17
tags:
---

文/大大大大峰哥



![锤子时钟](http://upload-images.jianshu.io/upload_images/925416-05839a7036682df3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->

----------------

## 来源

首先先说明这个代码出自于[SmartisanTime](http://www.jcodecraeer.com/a/opensource/2016/0918/6619.html)，这里面有许多开源的代码，以后我学会了一个同时在我时间充裕的情况下，我都会在这里分析源码，将我的心得与如何实现同时我遇到的一些问题进行与大家进行交流分析。

## 适合人群
Android的初学者，熟悉基本的自定义View。


## 会遇到的一些问题
###### 1.Paint为什么添加阴影失败？
答：需要关闭硬件加速，函数上面添加 **@SuppressLint("NewApi")** ，同时在函数内部添加 **setLayerType(LAYER_TYPE_SOFTWARE, null);** ，最小**API8**。

 ###### 2.为什么需要重载onMeasure方法？
答：这个之前我也不知道为什么，因为我也是菜鸟，知道的大牛就可以跳过了，不知道的我们就一起学习下，我在 [Android开发实践：为什么要继承onMeasure](http://ticktick.blog.51cto.com/823160/1540134)得到了答案，具体就是说，如果没有使用这个方法，那么在XML中的wrap_content就与match_parent没有区别。同时在后面需要调用**setMeasuredDimension(size, size)**将Widch与Height保存起来。

 ###### 3.在使用 calendar.setTime(new Date()) 方法后时间为什么与我国时间不匹配？
答：因为我们通过这个方法得到是**“标准时间”**，而这个标准的时间为美国的时间，所以在运用这个方法之后，将小时加上八个小时就是我们的北京时间。


##  我的代码
https://github.com/qq282629379/SmartisanTime

##  作者有话说
我也是刚刚入Android的大门，以后可以与各位互相学习，共同进步，这次解析的代码是一个比较容易的代码，我会慢慢循序渐进的，同时大家下载之后也可以提出自己的问题，在我能力范围内，我都会尽我所能，同时希望大家对我的代码指出问题。