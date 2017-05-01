---
layout: android
title: 简单描述Android异步原理
date: 2017-05-02 00:16:01
tags: 多线程
---

**文/大大大大峰哥**

## 绪
本来是想着继续写Service（二），看了很多博文也不知道怎么写下去，今天索性宿舍停电看了看《第一行代码》第一版的Serivice部分，作者是首先讲述的异步在慢慢的托出Serivice，所以今天就根据看的书，做点笔记，同时将知识点划分出来，日后自己看也好整理学习。



<!-- more -->

## 概述

异步处理工作是通过四个部分组成：  `Message`,`Handle`,`MessageQueue`,`Looper`。

**Message**：Message是消息的意思，是`进程与进程`之间传递的消息，主要是用来在两个线程中做信封的角色，同时信封里可以存int，object类型的值。 

**Handle**：Handle是处理者的意思，在我看来这个是装信箱的箱子，有两个方法，一个是sendMessage这个就是发送信件的箱子，还有一个就是handleMessage这个就是接受箱子，在里面通过Message的数据来进行随意处理。

**MessageQueue**：MessageQueue是消息队列，就是信件堆放的队列，根据队列的先进先出的规则，依次被Looper提出来给Handle。   

> 注意：每一个线程只会有一个MessageQueue对象。

**Looper** ：文中解释这个是MessageQueue的管家，调用loop方法后进入无限循环，发现MessageQueue中存在消息就按照队列规律提取出来，传递到handleMessage中。

> 注意：每个线程只会出现一个Looper。

## 例子

Android的UI更新必须在主线程中更新，如果我们更新UI在子线程中更新就会报错，但是如果我们使用异步处理就可以这样实现，下面我列出代码，根据代码来描述。

```
package com.example.tangzhifeng.studydemo;

import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
    Button btn;
    TextView tv;
    Handler mHandler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            tv.setText("更新Text");
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        btn= (Button) findViewById(R.id.button);
        tv= (TextView) findViewById(R.id.text);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(){
                    @Override
                    public void run() {
                        super.run();
                        Message message=new Message();
                        mHandler.sendMessage(message);
                    }
                }.start();
            }
        });
    }
}
```

## 总结

在该例子中，我们的Handle是在主线程中创建的，然后我们在子线程中创建了Message，同时我们的将这个Message投到了`主线程`的Handle sendMessage中，然后进入了`主线程`的MessageQueue，`主线程`的Looper发现`主线程`的MessageQueue有一条Message于是调用了dispatchMessage，然后Message来到了主线程的handleMessage，然后更新的UI。


![异步处理工作原理](http://upload-images.jianshu.io/upload_images/925416-2b1263e7c0f5da56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在我之前看书的时候发现，**Message是在子线程创建的，主线程的Handle在子线程中使用了sendMessage方法，那到底进入了那一个MessageQueue中呢？是主线程的，还是子线程的呢？**后来在一个Android技术群中，有个大牛告诉我是看Handle是在哪个线程创建的就是在哪个线程。这个地方其实仔细想想其实就是这样的，可能是读傻了。。。所以我特意标记了是哪个线程的XXX，这样方便我看，同时也希望帮助和我有同一个问题的人。

## End

希望大家多多指点，有问题指出来，互相帮助互相学习。