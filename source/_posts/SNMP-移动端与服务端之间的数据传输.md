---
layout: android
title: SNMP 移动端与服务端之间的数据传输
date: 2017-05-02 00:15:34
tags:
---

 **文／大大大大峰哥**

###### 写作目的
现阶段的工作需要完成Zabbix管理Android，那么是通过SNMP进行数据传输，当前博主也是在探索，大家可以一起探讨交流。
###### 工作环境

Windows10、Android5.1.1系统、Ubuntu12

<!-- more -->

--------------------
## 1、移动端与服务端互相通信

###1.1 目的
证明移动端可以与服务端进行通信，所以先去各大手机应用商城去看看别人是否有写出直接可以通信的APP。

### 1.2 前提

需要配置Windwos下的SNMP的简易服务器。**（暂时不知道这个步骤是否一定要）**

#### 操作流程

![开始进入控制面板](http://upload-images.jianshu.io/upload_images/925416-ca633f471de586d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![进入程序](http://upload-images.jianshu.io/upload_images/925416-f35733896895fa74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  ![启动或关闭Windows功能](http://upload-images.jianshu.io/upload_images/925416-8f98c93774f034aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)   ![将这WMI SNMP打上勾](http://upload-images.jianshu.io/upload_images/925416-1b88e5a4b88cc9f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)   ![我的电脑右键 **管理** ](http://upload-images.jianshu.io/upload_images/925416-dcfef79f2d9f7c53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  ![右键**属性**](http://upload-images.jianshu.io/upload_images/925416-2326a059100dbd17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  ![点击添加](http://upload-images.jianshu.io/upload_images/925416-ba782dd40c5ffd85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  ![按照我这个样式点击添加](http://upload-images.jianshu.io/upload_images/925416-737408860c71fc48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.3过程
通过这个工具（****Snmptester.zip****）在Windows10上可以发送指定的OID给手机SNMP管理工具（****SNMP Agent****），手机成功的接受到数据，并将需要传输的OID返还给Windows。

### 示例
![Snmptester操作界面](http://upload-images.jianshu.io/upload_images/925416-e4d98e1409328ba6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 )

![Android发送数据后收到数据的反馈](http://upload-images.jianshu.io/upload_images/925416-b0f984be168d23ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里移动端暂时通过豌豆荚下载的一个小工具 ****SNMP Agent**** ，主要是通过实现一个连接来证明手机与Windows之间是可以实现SNMP通信的。

这样我们就证明了 移动端是可以与电脑进行一个SNMP的数据连接。

---------------------------



## 2、运用SNMP4J进行连接服务器
目前我是通过Android去与服务端通信，当然我是借助其他的包来帮助我与服务器通信，我在网上找了很多资料，发现有一遍[博文](http://www.cnblogs.com/xdp-gacl/p/4187089.html)成功通过SUMP4J来实现了。
### 2.1 下载SNMP4包
[SNMP4下载界面](http://www.snmp4j.org/html/download.html)

![我是通过迅雷下载的，Chrome下载没有反应](http://upload-images.jianshu.io/upload_images/925416-ff3e10175587aa4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![解压后这两个就是jar包了](http://upload-images.jianshu.io/upload_images/925416-64f42c3a0279ce6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.2 JAVA实现SNMP

###### 编译环境
eclipse jdk8 snmp4j-2.5.0.jar snmp4j-2.5.0-javadoc.jar

在这里主要是以[孤傲苍狼博文](http://www.cnblogs.com/xdp-gacl/p/4187089.html)代码为主，博主还没有具体去分析代码的意义。我这里只是在运用，而没有去想是如何实现的，我上传一份按照他思想的[**源码**](http://pan.baidu.com/s/1hrVcsVM)。

![比较关键的就是OID与Agent地址，Angent地址需要自己修改！！！](http://upload-images.jianshu.io/upload_images/925416-4a0e6adfcab1bd69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![我改成了v1版本是因为**Snmptester**软件只支持v1，**Snmptester**高于v1就会报错，暂时不知道是什么原因](http://upload-images.jianshu.io/upload_images/925416-8f91e93d4fb1a2e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上操作，博主均已通过测试。

### 2.3 JAVA实现自定义的OID传输
###### 参考资料
[萌萌的It人 www.itmmd.com](http://blog.csdn.net/dyllove98/article/details/7720060)

在这篇博文中，详细的讲解了传递String类型的数据与传递字节数组。
主要内容：VariableBinding方法中运用到了Variable类型，而Variable类型中不一定要采用OctetString，里面有一个用Integer32的实现方式。

###### 对照组A

![通过Java代码发送了一个320](http://upload-images.jianshu.io/upload_images/925416-eb248fc1da823fa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![Android控制台上记录的值
](http://upload-images.jianshu.io/upload_images/925416-1bab0660ba882a5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###### 对照组B

![通过Java代码发送了一个319](http://upload-images.jianshu.io/upload_images/925416-b55b1fc74292dc2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Android控制台上记录的值](http://upload-images.jianshu.io/upload_images/925416-a2a607cb8aac629e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


对照组中的A与B，A中android控制台上记录的值为140，B则为13f，然后十六进制的140转换为十进制为320，13f则为319

**所以在参考资料中的博文是可行有效的。**

## 3、移动端SNMP数据传输
######  参考资料
[linux snmptrap的发送与接收](http://blog.chinaunix.net/uid-29068508-id-3821872.html) 、[Android-SNMP](https://github.com/qq282629379/Android-SNMP)


### 3.1 实现snmptrap接受发送
首先需要在Linux上实现一个trap的命令接受与发送，这样才可以知道snmptrap是否可行。这里面因为我是在mac上面做的验证，traphandle这个命令并没有，但是依旧可以传递数据，只是oid没有创建，详细还是看我参考资料。

```
配置代码：
Conf代码 ：authcommunity execute,log,net public 
traphandle .1.3.6.1.4.1.2021.251.1 /root/traptest/test.pl 
其中authcommunity是为了设置所有用户的访问权限：可执行，记录，传递。
设置traphandle(即收到.1.3.6.1.4.1.2021.251.1类OID信息时，执行test.pl)。
test.pl的内容：
#!/usr/bin/perl  
use strict;  
my $file="file.trap";  
open(HANDOUT,">>./$file");  
while()  
{      
print HANDOUT "$_";  
}  
```


![发送trap](http://upload-images.jianshu.io/upload_images/925416-0a57629007fefa27.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![接收端](http://upload-images.jianshu.io/upload_images/925416-a151582af8e0dbe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)