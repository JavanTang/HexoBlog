---
layout: android
title: Android Notification学习
date: 2017-05-02 00:16:31
tags: Notification
---

**文/大大大大峰哥**

## 概述

在我们Android开发过程Notification是一个使用率较高的组件，我们有理由仔细学习它。使用场景一般是在程序进入了后台的时候我们才需要用到通知。


## 创建通知

1. 首先我们需要创建一个NotificationManager来对通知进行管理。NotificationManager通过调用`Context.getSystemService(Context.NOTIFICATION_SERVICE)`。
2. 创建Notification对象，通过`Notification.Builder`创建，必须包含`setSmallIcon()`小图标、 `setContentTitle() `标题、`setContentText() `详细文本。
3. 然后调用NotificationManager的notify方法传入两个参数：id与Notification对象。


<!-- more -->

## PendingIntent

在我们日常使用APP的时候，我们会发现通知栏的消息我们点击后，会进入一个新的Activity，这个就是通过PendingIntent实现的。PendingIntent与Intent类似，但是它是一个延时执行的Intent。

**创建方法**

使用PendingIntent我们是通过PendingIntent自身的`getActivity`方法，需要传入四个参数：`Conten`、第二个参数一般用不到`传0`即可、第三个参数为`Intent`对象、第四个参数为`PendingIntent行为`。

**PendingIntent行为：**

- FLAG_ONE_SHOT：Flag indicating that this PendingIntent can be used only once.
- FLAG_NO_CREATE：Flag indicating that if the described PendingIntent does not already exist, then simply return null instead of creating it.
- FLAG_CANCEL_CURRENT：Flag indicating that if the described PendingIntent already exists,the current one should be canceled before generating a new one.
- FLAG_UPDATE_CURRENT：Flag indicating that if the described PendingIntent already exists, then keep it but replace its extra data with what is in this new Intent.
- FLAG_IMMUTABLE：Flag indicating that the created PendingIntent should be immutable. This means that the additional intent argument passed to the send methods to fill in unpopulated properties of this intent will be ignored.


## 手动取消

NotificationManager.cancle(id)


## 通知的高级技巧

- 设置提醒声音：`sound`属性。
- 设置震动：`vibrat`属性，传入long数组，前一个为`静止`时长，后一个为`震动`时长。
- 设置LED灯，需要使用`ledARGB`，`ledOnMS`，`ledOffMS`以及`flags`属性来实现。

> **注意：**这里需要写入震动的权限。


**△例子（实现绿色灯一闪一闪）：**
```
notification.ledARGB=Color.GREEN;
notification.ledOnMS=1000;
notification.ledOffMS=1000;
notification.flags=FLAG_SHOW_LIGHTS;
```

**使用默认提醒**
如果不想麻烦，就直接使用系统默认的提醒，我感觉也不挺不错的。

```
notification.defaults=Notification.DEFAULT_ALL;
```