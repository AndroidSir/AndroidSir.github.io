---
layout: post
title:  "todo-mvp"
date:   2017-02-15
categories: 知识点
tags: MVP 
---

* content
{:toc}

鉴于自己写的Android代码太杂太乱，这里学习MVP代码结构的官方例子，自己不懂的地方在这里记录。




## 涉及到的新知识

[Android MVP 详解（上）](http://www.jianshu.com/p/9a6845b26856)

[谷歌官方MVP框架源码解析之 TODO-MVP](http://www.jianshu.com/p/8b81493d1297)

[解读Android官方MVP项目单元测试](http://www.jianshu.com/p/cf446be43ae8)

[Android官方MVP架构示例项目解析](http://mp.weixin.qq.com/s?__biz=MzA4MjA0MTc4NQ==&mid=404088059&idx=3&sn=78dafacbca09b0d7345344c3eef24aff#rd)

在导入`todo‑mvp`工程的时候，编译报错，通过[这里](http://blog.csdn.net/u013164293/article/details/53318285)解决了。

AppBarLayout：[玩转AppBarLayout，更酷炫的顶部栏](http://www.jianshu.com/p/d159f0176576)

Toolbar：主要是用来替换ActionBar的

CoordinatorLayout:[CoordinatorLayout的使用如此简单](http://blog.csdn.net/huachao1001/article/details/51554608)

FloatingActionButton:

NavigationView:[Android 自己实现 NavigationView Design Support Library(1)](http://blog.csdn.net/lmj623565791/article/details/46405409)

Android VectorDrawable与SVG:[Android VectorDrawable与SVG](http://blog.csdn.net/xu_fu/article/details/44004841)

actionBar.setHomeAsUpIndicator:[Android开发 更改返回按钮的图标](http://www.tuicool.com/articles/eEVzY3)

menu:

Guava:

SwipeRefreshLayout：

#首页面 

## View层与Presenter的定义

**View层原始接口**的定义如下图，可以看出该接口的定义使用了类型参数，即View层具备知道其引用的Presenter是谁的功能。

<div  align="left">    
<img src="http://a2.qpic.cn/psb?/V11DxkGh190yEc/GztdYKqYud9yT0baC4*6uHtqUVpJaL5O1ZOIqv68vrI!/b/dB4BAAAAAAAA&bo=UwG6AAAAAAADB8o!&rf=viewer_4" width = "339" height = "186" />
</div>

**Presenter层原始接口**的定义如下，可以看出该接口只定义了作为中间人开始工作的职责。

<div  align="left">    
<img src="http://a2.qpic.cn/psb?/V11DxkGh190yEc/**GaPSTr.wjIaAp*rKA4ArrzHpjYyQo5ntA65Z1yOb8!/b/dB4BAAAAAAAA&bo=MgGfAAAAAAADB44!&rf=viewer_4" width = "306" height = "159" />
</div>

作为**View层与Presenter层的统领**，工程为应用首界面（Tasksxxx）定义了`TasksContract`接口，该接口定义如下：

<div  align="left">    
<img src="http://a2.qpic.cn/psb?/V11DxkGh190yEc/*ftW3hDxNWTKS587Jaym4xvcasNRmzngHh810ZJB*k4!/b/dB4BAAAAAAAA&bo=hwFpAgAAAAADB88!&rf=viewer_4" width = "391" height = "617" />
</div>

从接口结构图可以明确看出**View层**需要完成的功能以及**Presenter层**需要完成的功能。可以发现一个规律，View层接口的定义绝大部分的方法名都以`show`开头，因为View层本身定义的就是视图层的交互，要么显示，要么隐藏。这算是一个建议，以后可以参考。

由于首页可以显示的任务列表有三种情况：显示全部、显示已完成、显示待完成。因此工程又定义了一个枚举类，来区分这三种情况，如下图所示：

<div  align="left">    
<img src="http://a3.qpic.cn/psb?/V11DxkGh190yEc/bs5RkNVL5NYW69CGOcScuVvoMh96abWdrUOJIMD9qTw!/b/dB8BAAAAAAAA&bo=DAIGAgAAAAADByg!&rf=viewer_4" width = "524" height = "518" />
</div>  

## View层的实现

在工程中，`TasksFragment`做为View层的实现类，实现了View接口定义的抽象方法。其中重点看一下`showTasks(List<Task> tasks)`方法的实现：

<div align="left">
<img src = "http://a1.qpic.cn/psb?/V11DxkGh190yEc/j9l4qePqa5x5gGUO9yJCw30UoW1jwuMijz.3*Y9JRgQ!/b/dCABAAAAAAAA&bo=lQEGAQAAAAADB7E!&rf=viewer_4" width="" height=""/>
</div>

在看到这个实现的时候，我就在想，如果传入的参数tasks不是null但是empty的时候怎么办？后来发现在Presenter中调用View层的showTasks的时候，已经保证所传参数非null非empty了。进一步理解了View层只负责展示，不负责数据的各种判断，而应将各种判断置于Presenter层进行处理。

`TasksFragment`中还为ListView定义了一个适配器，与平常使用适配器的方式不同。平常使用适配器的时候，数据源一旦赋值，当数据元素发生改变的时候，调用Adapter对象的`notifyDataSetChanged();`方法即可。但是工程中则是直接为Adapter的数据集合成员变量切换数据容器。

最后需要说明一下，`TasksActivity`在工程中的作用。观察代码可以知道，`TasksActivity`主要负责创建View层的实现以及Presenter的实现。当然，因为需求原因，这里`TasksActivity`在创建Presenter的实例之后还调用了其中的方法。

View层持有Presenter实例的引用。

## Presenter层的实现

可以看到，`TasksPresenter`作为`TasksContract.Presenter`的实现类，持有View层的引用以及Model层的引用。而赋值操作是在构造函数中完成的，并且在构造函数中调用View层实例的方法，将自己的引用赋值给View层，代码如下图所示。

<div align="left">
<img src = "http://a3.qpic.cn/psb?/V11DxkGh190yEc/DvCLhckV3E92z9Uiz4SAmz4gBZT8nP8ppv.63ECMkiE!/b/dK0AAAAAAAAA&bo=uQM0AQAAAAADB60!&rf=viewer_4" width="" height=""/>
</div>