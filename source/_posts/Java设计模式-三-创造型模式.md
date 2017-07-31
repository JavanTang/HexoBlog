---
layout: java
title: Java设计模式(三)创造型模式
date: 2017-05-12 20:05:43
tags: [java,设计模式]
---

## 概述

前面两篇基本上是一个大纲与一些前人约定俗成的一些规则,然后在这些规则的制约下再细分为三大类型:`创建型`,`结构型`,`行为型`.今天来说说其中之一创建型.

创建型顾名思义是对类的创建过程的抽象化的模式,创建型模式共有五种:

1. 单例模式
2. 工厂方法模式
3. 抽象工厂模式
4. 建造者模式
5. 原型模式

其实其中每一个模式都可以重开一章细说,但是因为时间已经篇幅的原因,暂时不打算细说,如果有需要可以留个言,我也会独立开一篇来和大家共同探索,共同成长.

<!-- more -->

## 单例模式

### 定义

一个类只有一个实例,而且`自行`实例化并向整个系统提供这个实例.在我们日常写项目的时候,数据库操作db,我们通常就会使用单例模式,或者我们统计一个Web页面的访问人数等等例子.单例模式的表现形式一般有两种表示形式:

- 饿汉式单例类:类加载时,就进行实例化.
- 懒汉式单例类:第一次引用类时,才进行对象实例化.

#### 饿汉单例模式

```java
public class HungryMan{
  static HungryMan hungry=new HungryMan();
  //私有的构造函数
  private HungryMan(){
    
  }
  public static HungryMan getInstance(){
    return hungry;
  }
}
```

当我们加载该类的时候,这个hungry对象会直接被初始化,单例模式的最大的特点就是`私有`的构造函数.

####  赖汉单例模式

```java
public class LazyMan{
  static LazyMan lazyMan;
  private LazyMan(){
    
  }
  synchronize public static LazyMan getInstance(){
    if(lazyMan==null){
  		lazyMan=new LazyMan();
    }
    return lazyMan;
  }
}
```

>  注意:在这里之所以使用synchronize是防止在多线程中出现多个lazyMan.

懒汉模式的实例化是发生在`getInstance`里面的,其实最让我不理解的是这两个命名,大家就这么理解,先实例化的就是饿汉,急急忙忙的进行实例化,懒汉就是不到最后一刻绝对不进行实例化.

从`资源利用率`上面来说,饿汉式单例类比赖汉式单例类要差一些,但是从`速度和反应时间`上来说,饿汉模式比懒汉式要好些,大家自我取舍.



#### 单例模式优点

- 在全局访问时,优化了对资源的访问.


- 只有一个实例那么减少了对一个对象频繁地被创建,销毁的内存开支.


#### 单例模式缺点

- 无法创建子类,所以扩展困难
- 测试不方便
- 与单一职责原则有冲突

### 实例

我们写一个Web记录访问人数的项目,我们将记录人数的类使用单例模式,然后创建两个线程,然后让两个线程持续访问来演示单例模式.

```java
package single;

/**
 * Created by tangzhifeng on 2017/5/12.
 */
public class Web {
    int accessSize;
    public static final Web sWeb=new Web();

    private Web(){
        accessSize=0;
    }

    public static Web getInstance() {
        return sWeb;
    }

    synchronized void visit(){
        accessSize++;
        System.out.println("第"+accessSize+"次访问");
    }

}
```

```java
package single;

/**
 * Created by tangzhifeng on 2017/5/12.
 */
public class VisitThread extends Thread{


    @Override
    public void run() {
        while(true) {
            Web.getInstance().visit();
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
package single;

/**
 * Created by tangzhifeng on 2017/5/12.
 */
public class Test {
    public static void main(String[] args) {
        VisitThread visitThread1 = new VisitThread();
        VisitThread visitThread2 = new VisitThread();
            visitThread1.start();
            visitThread2.start();
    }
}
```

![](http://ww1.sinaimg.cn/large/006tKfTcgy1ffiy9efaroj31kw0btjue.jpg)

这里有两个两个线程一起访问Web,因为是单例模式,所以可以统一对访问次数进行统计.



## 工厂模式

### 定义

定义一个用于创建对象的接口,让子类决定实例化哪个类.工厂模式可以分为`简单工厂`,`工厂方法`和`抽象工厂模式`:

- 简单工厂模式中,工厂类管理创建产品,初始化的过程都在工厂类中,从我们创建产品类的角度上来说,是符合"`开闭原则`"的,但是对"`开闭原则`"的支持力度又不够,当有新的产品加入,必须修改工厂类.
- 工厂模式是在简单工厂模式的基础上的优化版,从简单工厂模式中的一个工厂变成了多个产品工厂,每个产品工厂对应一个产品,这样的就在原来的`简单工厂`的基础上完善了"开闭原则",有新的产品加入也不会修改任何一个类.
- 抽象工厂模式顾名思义理所应当是工厂模式中最`抽象`的,它与工厂模式的区别,主要是由工厂模式中的`一个`产品族变成了`多个`产品族.

> 具体大家看后面的例子,大家看完例子再回过头回味一下这三种定义,如果大家有一些不一样的理解,我希望大家回复留言,我希望和大家一起探讨探讨.

### 简单工厂模式

![](http://ww3.sinaimg.cn/large/006tNbRwgy1ffr1uxcfenj30o80ejq30.jpg)

```java
class Factory{
  Car create(String type){
    if(type.equals("Benz")){
      return new BenzCar();
    }eles if(type.equals("BMW")){
      return new BMWCar();
    }
  }
}
```

上面的例子是一个简单工厂模式创建奔驰车和宝马车的UML图.有一个基类Car,BenzCar与BMWCar继承Car,然后我们的创建通过Factory来创建,通过传递的type来区别创建是创建宝马还是奔驰.

在简单工厂模式中有一个Factory类,这个类控制创建Car的类型,但是我们会发现如果当我们多一个Car的种类,就需要修改Factory中的代码,这就不符合"开闭原则".

### 工厂模式

![](http://ww2.sinaimg.cn/large/006tNbRwgy1ffr633a5ikj30o80fvweq.jpg)

还是一样的例子,但是这里的Factory也变成了基类,这里我们奔驰与宝马的创建需要有一个实体奔驰或者宝马的工厂类,这样一一对应互不干扰.同时也对后期的扩展毫不影响.

在这里大家知道与简单工厂模式的不同了么,就是每一个类型的Factory对应一个类型的产品,大家仔细想想这样有什么好处?是不是当我们需要添加一个类型,可以直接扩展而不需要修改任何代码,这样比起简单工厂模式是不是好了很多,这样的结构方便我们后期项目的扩展.

### 抽象工厂模式

![](http://ww2.sinaimg.cn/large/006tNbRwly1ffr71a9zm7j30v10id0tr.jpg)

这个例子是我找的一本书上的例子貌似是DOTA,有两方`天灾军团`与`近卫军团`,每个军团都对应同类型的英雄,但是不同的英雄名,不同的技能,这里就用到了抽象工厂模式,抽象工厂模式我们千万不要用字义去理解与工厂模式的区别,首先我个人觉得工厂模式与抽象工厂模式就一点点联系.大家先自己想想这种模式的优点和缺点,然后再看后面的.

这里两个工厂实体类都实现了三个获取方法,抽象工厂最大的优势就是针对多个产品族,比如此时此刻又多了一个人类军团工厂,部落军团工厂,这时候只要继承`英雄工厂`就实现了扩展.但是当我们又多了一类英雄类型,比如魔力英雄,战斗英雄,那我们就要改基类`英雄工厂`以及所有它的子类.

当抽象工厂的产品族变为一种就退化成了工厂模式,这也就是我认为仅有的联系.



## 建造者模式

![](http://ww2.sinaimg.cn/large/006tKfTcgy1ffxpsr8vppj30rm0b8q30.jpg)

### 定义

在我写这个模式的时候,我主要在思考它与工厂模式的不同,同样都是创造类型的模式,他们之间主要的不同是:在工厂模式中,产品类的属性都是在我产品类中实现;而建造者模式中,产品类的属性在建造者中实现,然后在导演类中选择`装配顺序`.

> 装配顺序也会影响产品类,致使产品类不一样.

### 实例

这里取一个书上的例子,生产手机的例子,但是这里的例子没有突出`装配顺序`影响产品类.

