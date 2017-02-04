---
layout: post
title:  "Android开发中VersionCode与VersionName概述"
date:   2017-02-04
categories: 知识点
tags: Android VersionCode VersionName
---

* content
{:toc}

Google为APK定义了两个关于版本属性：VersionCode和VersionName，他们有不同的用途。VersionCode与VersionName即可以作为清单文件中`<manifest>`标签的属性进行赋值，又可以在主module的`build.gradle`文件中给值。本文就这两个概念进行学习。




## 官方文档

[参考至](https://developer.android.google.cn/guide/topics/manifest/manifest-element.html)

android:versionCode

内部版本号。该数字仅仅用于判定不同版本之间哪个是最新的，数字越大代表版本越新。这个版本号不展示给用户看，展示给用户看的版本号由`versionName`属性设置。内部版本好的值必须是一个整形，比如100。我们可以定义它为任意的数值，只要保证该值是递增的即可。例如

android:versionName

该版本号用于展示给用户。该属性可以被设置为一个字符串或者一个字符串资源得引用。该字符串除了展示给用户没有其他目的。versionCode属性值才是真正的系统内部用于使用的。

## 通俗解释

[参考至](http://www.2cto.com/kf/201312/266368.html)

VersionCode：对消费者不可见，仅用于应用市场、程序内部识别版本，判断新旧等用途。
VersionName：展示给消费者，消费者会通过它认知自己安装的版本，下文提到的版本号都是说VersionName。 

**版本**：大家在使用软件和应用时，都会涉及到版本的概念，大家都知道的，比如Win XP，QQ2012，小米桌面1.6。之所以会有版本，主要是因为软件产品一直在发展、变化的。版本的概念可以帮助消费者识别不同时期的产品。 

而展现在消费者面前的版本，和开发者内部使用的通常是不同的版本。开发时通常会使用数字作为标志，比如`6.1.7600.16385`其实是Win 7第一个正式版的版本号，而Win 7 SP1的版本号是`6.1.7601.17514`，这样长长一串数字对消费者毫无意义，所以在产品发布时通常会起一个更容易懂的版本。下文中会把Win 7这样的用于展示的版本叫做[VersionName]，6.1.7601.17514这样用于程序标识的版本叫做[VersionCode]。

早年因为软件主要自己负责自己的分发、升级等方面，所以版本号也相当自由，各家都有不同的规范。但是近年来移动设备崛起，App Store这样的应用商店集中分发成了主流。以升级为例，应用商店会负责检查消费者手机上应用的版本，并和商店里面最新的版本比较，如果商店里面的版本比较新，消费者手机上的版本比较旧，就会提醒消费者升级。

这就涉及到如何识别新、旧的问题。 对于计算机来说，最可靠的判断方式就是数字，数字有很多好处：程序容易判断、格式简单不容易出错、肉眼容易识别等。所以Google要求每个应用都要在APK安装包中记录这个安装包的[VersionCode]，只要拿到这个APK文件，就可以知道它对应的[VersionCode]是多少，应用商店就会以这个[VersionCode]为准，来判断版本。安装包的[VersionCode]数字越大就越新。这样开发者在开发过程中，每有一个新版本只要加大一点这个数字就可以了。比如第一个版本的[VersionCode]是1，第二个版本是2。因为开发者可能每天可能会产生多个没有发布的版本，所以这个数字会增长的很快。

经过一段时间的开发，这个数字会变得比较大，比如16385，这时对一个消费者，这样的数字其实不太具有可识别性，比如说Win 16385和Win 17514在传达信息方面效果并不好，不利于产品的市场推广。因此Google也支持在AKP安装包内记录[VersionName]，你可以叫Win 7、Win Vista都没问题，可以满足市场、传播方面的需求，这样[VersionName]其实不具备比较新、旧版本的能力，只是用来展示给消费者看的。  

开发者常会遇到以下一些问题:

* 同一个版本号，对应了多个VersionCode怎么办？

比如说新版本发布之后，某个商店反馈说存在xxx问题，需要修复、定制等等操作，于是商务找工程师出了个新版本，考虑到是小版本升级，版本号没变化，但是VersionCode已经变了。 
可能遇到的问题：如果这个新版只在部分商店上线，就会出现都是3.1版，A商店的版本其实比B商店的新。已经安装了新版本的用户，还会被提示升级，这时候用户会困扰，为什么我装了3.1还要升级到3.1？部分商店为了最新会抓包，导致渠道包流窜，影响运营监控和分析。
解决方案：a.版本号应该和VersionCode一起涨，而且一旦发布新版本，就在所有渠道上架新版。

* 发布了一个VersionCode错误的版本怎么办？

有时候因为工程师不小心，发布了一个VersionCode过大的版本，比如1.1.1.20版本的VersionCode写成了111，而1.1.1.27版本的VersionCode写成了11127，但是后面发布1.1.2版希望延续旧的VersionCode，用112。 
可能遇到的问题：1.1.1.27版的用户将无法获得1.1.2版本的升级，因为在程序看来1.1.1.27版本是比较新的，同时，已经使用了1.1.2版本的用户，可能会收到旧版本的升级提示，比并降级回旧版
解决方案：其实很简单，因为VersionCode对最终用户是不可见的，只要增加就好了，上文的例子，新版VersionCode直接取11200就齐活了。

* 发出去的应用有Bug要换回旧版，怎么操作？

发布了一个有Bug的版本，好捉急 偶尔会遇到版本已经发布了，第二天突然发现，糟糕，有Bug，用户开始骂了！于是商务同学到各家市场要求退回旧版本。 
可能遇到的问题：已经升级到有Bug版本的用户是无法回滚到旧版的，因此这样直接退回旧版本的方式对这些热心升级的用户是非常不负责任的。而且人肉召回的力度实在有限，这个有Bug的版本一定会流传的。
解决方案：最好是不要浪费时间退回旧版，赶紧修复Bug发个新版本(记得加VersionCode)，如果Bug比较棘手，暂时无法修复，只能退回旧版本，这时建议把旧版本的VersionCode改大一些后，提交新版本，这样可以保证所有用户都能下载/升级到一个相对可靠的版本。

## 设值及代码取值

[参考至](http://www.cnblogs.com/libao/archive/2012/11/15/2771415.html)

在清单文件中设值：

	<?xml version="1.0" encoding="utf-8"?>
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	    package="com.xxx.xxx"
	    android:versionCode="2"
	    android:versionName="1.1">
	    ......

当app需要校对版本的时候怎样读取这个值？

	PackageManager pm = context.getPackageManager();//context为当前Activity上下文 
	PackageInfo pi = pm.getPackageInfo(context.getPackageName(), 0);
	version = pi.versionName;