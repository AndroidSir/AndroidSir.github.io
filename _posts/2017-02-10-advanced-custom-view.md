---
layout: post
title:  "自定义View"
date:   2017-02-10
categories: 知识点
tags: customview 
---

* content
{:toc}

Android中一次完整的事件传递主要包括三个阶段，分别是事件的分发、拦截和消费。




## 触摸事件的类型

触摸事件对应的是`MotionEvent`类，事件的类型主要有如下三种：

1. ACTION_DOWN
2. ACTION_MOVE
3. ACTION_UP

## 事件传递的三个阶段

1. 分发（dispatch）：事件的分发对应着`public boolean dispatchTouchEvent（MotionEvent e）`方法：

	* 返回值为true标识事件被当前视图消费掉，不在继续分发事件；
	* 返回`super.dispatchTouchEvent（）`标识继续分发该事件。

2. 拦截（intercept）： 如果当前视图是ViewGroup及其子类，则会调用`public boolean onInterceptTouchEvent（MotionEvent e）`方法判定是否拦截该事件(该方法只在ViewGroup及其子类中才存在，在View及Activity中是不存在的)：

	* 返回false或者`super.onInterceptTouchEvent`表示不对事件进行拦截，需要继续传递给子视图；
	* 返回true表示拦截这个事件，不继续分发给子视图，同时交由自身的`ouTouchEvent`方法进行消费。
	
3. 消费（consume）：事件的消费对应着`public boolean onTouchEvent(MotionEvent event)`方法：
	
	* 返回true表示当前视图可以处理对应的事件，事件不会向上传递给父视图；
	* 返回false表示当前视图不处理这个事件，事件会被传递给父视图的onTouchEvent方法进行处理。

在Android系统中，拥有事件传递处理能力的类有以下三种：

* Activity:dispatchTouchEvent(**分发**) onTouchEvent（**消费**）
* View:dispatchTouchEvent（**分发**） onTouchEvent（**消费**）
* ViewGroup:dispatchTouchEvent（**分发**） onTouchEvent（**消费**） **onInterceptTouchEvent（拦截）**

## 自定义View的属性

在`res/values/`下建立一个`attrs.xml` ，在里面定义我们的属性和声明我们的整个样式。

	<?xml version="1.0" encoding="utf-8"?>  
	<resources>  
	  
	    <attr name="titleText" format="string" />  
	    <attr name="titleTextColor" format="color" />  
	    <attr name="titleTextSize" format="dimension" />  
	  
	    <declare-styleable name="CustomTitleView">  
	        <attr name="titleText" />  
	        <attr name="titleTextColor" />  
	        <attr name="titleTextSize" />  
	    </declare-styleable>  
	  
	</resources>  

format是值该属性的取值类型，共有：string,color,dimension,integer,enum,reference,float,boolean,fraction,flag
