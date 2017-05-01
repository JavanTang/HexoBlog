---
layout: android
title: AsyncTask使用笔记
date: 2017-05-02 00:16:17
tags: [AsyncTask,多线程]
---

**文/大大大大峰哥**

## 序
AsyncTask是基于异步消息处理机制，只是Android为了方便我们使用，将它封装了起来，这让我们可以更加简单便捷的使用，可以轻松地从子线程切换到主线程。如果想简单了解异步消息处理机制可以看我另外一篇博文[简单描述Android异步原理](http://www.jianshu.com/p/f976ee81c913)。

<!-- more -->

## 使用
AsyncTask是一个抽象类，使用它需要继承它，继承的时候需要注意它拥有三个泛型参数：`Params`，`Progress`，`Result`。

**Params**：需要`传入`的参数，用于后台任务中使用，如果你不需要传入数据可以将该设置为`Void`。

**Progress**：后台任务执行时，往往需要了解后台任务的进度，这个参数自行设定成为你需要反映进度的数据类型，比如我需要将进度用Integer返回等等。

**Result**：当任务执行完毕后，则使用该类型作为`反馈类型`，比如使用Boolean类型，那么你可以返回true告诉主线程，我成功运行。

## 方法

在我们继承AsyncTask后，我们需要了解几个重用的重写方法。

**onPreExecute()**：在我们使用AsyncTask需要`初始化`操作的时候，我们可以重写该方法并在里面初始数据以及UI。比如我们可以初始化一个下载进度条。

**doInBackground(Params ...)**：这个是当你继承AsyncTask`必须`重写的方法，该方法所有代码均在`子线程`中运行，一般是用来处理`耗时`操作。比如我们在这方法向网络获取数据。
> 注意：因为在子线程中运行，那么就不能做更新UI的操作。

**onProgressUpdate(Progress ...)**：在该类中调用publishProgress方法后，才会运行这个方法中的代码，通过传入的Progress类型后我们可以处理并更新UI。

**onPostExecute(Result )**：当我们后台任务（doInBackground）执行完毕之后，return 传递的值进入onPostExecute方法中，向我们传递任务执行结果，比如我们可以关闭下载进度条对话框，并且提示下载成功或者失败。

##代码
学习AsyncTask光靠理论说是不行的，所以需要看代码来辅助理解，在看完之后自己敲一遍才是自己的东西，下面是做了一个模拟下载的功能。

**（一）MainActivity.java**
```
package com.example.deme2;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {

    Button mButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mButton= (Button) findViewById(R.id.button);
        mButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new MyAsyncTask(MainActivity.this).execute();
            }
        });


    }
}
```


**（二）MyAsyncTask.java**
```
package com.example.deme2;

import android.app.AlertDialog;
import android.app.Dialog;
import android.content.Context;
import android.os.AsyncTask;
import android.util.Log;
import android.widget.Toast;

import static android.content.ContentValues.TAG;

/**
 * Created by TangZhiFeng on 2016/12/14.
 */

public class MyAsyncTask extends AsyncTask<Void, Integer
        , Boolean> {


    Context mContext;
    AlertDialog mAlertDialog;
    int mDownProgress;

    public MyAsyncTask(Context context) {
        mContext = context;
    }

    /*
        * 这个函数一般是完成一些初始化操作。
        * */
    @Override
    protected void onPreExecute() {
        mDownProgress = 0;
        mAlertDialog = new AlertDialog.Builder(mContext)
                .setTitle("下载进度")
                .setMessage(mDownProgress + "")
                .create();
        mAlertDialog.show();
    }


    /*
    * 这个函数一般都是处理一些耗时操作，同时这个函数
    * 是在子线程完成，不能进行UI更新操作
    * */
    @Override
    protected Boolean doInBackground(Void... params) {

        try {
            while (true) {
                mDownProgress = doDownload();//模拟下载过程
                if (mDownProgress > 100) {
                    break;
                }
                Thread.sleep(1000);
                publishProgress(mDownProgress);
                Log.i(TAG, "doInBackground: "+mDownProgress);
            }
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

        return true;
    }

    private int doDownload() {
        return mDownProgress+5;
    }

    /*
    * 调用publishProgress后，进入这个函数，可以进
    * 行UI操作
    * */
    @Override
    protected void onProgressUpdate(Integer... values) {
        mAlertDialog.setMessage("" + values[0]);
    }

    /*
    * 在后台任务执行完毕后调用该函数
    * */
    @Override
    protected void onPostExecute(Boolean aBoolean) {
        mAlertDialog.dismiss();//关闭对话框
        if (aBoolean) {
            Toast.makeText(mContext, "下载完成", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(mContext, "下载失败", Toast.LENGTH_SHORT).show();
        }
    }
}

```


## 总结

![MyAsyncTask流程图](http://upload-images.jianshu.io/upload_images/925416-99921d9d8db9083f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## End
希望大家多多指点，有问题指出来，互相帮助互相学习。