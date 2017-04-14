---
layout: post
title:  "MVP之View层非空"
date:   2017-04-14
categories: 手刃bug
tags: MVP 回调 异常
---

* content
{:toc}

本着以对自己代码负责的态度，我不断地重构着过去的代码，优化着目前的代码。在以MVP方式实现我的岛屿列表的时候发现一个问题：后台响应到达时移动端View层已经被销毁了，于是乎发生了空指针异常，于是乎就有了这篇问题解决日志。




## 问题回顾

Presenter层对象服务View层，对下使用Model层。若Presenter层一直持有View层对象的引用，很大可能会引发内存泄漏问题。因此考虑到让Presenter层仅仅持有View层的弱引用，降低发生内存泄漏的风险。Presenter层基类代码如下：

	public abstract class BasePresenter<T> {
	    protected Reference<T> mViewRef;//View接口类型的弱引用
	
	    public void attachView(T view){
	        mViewRef = new WeakReference<T>(view);//建立关联
	    }
	
	    //获取View层
	    public T getViewLayer(){
	        return mViewRef.get(); 
	    }
	
	    //判断是否与View建立了关联
	    public boolean isViewLayerAvailable(){
	        return mViewRef!=null&&mViewRef.get()!=null;
	    }
	
	    //解除关联
	    public void detachView(){
	        if(mViewRef!=null){
	            mViewRef.clear();
	            mViewRef = null;
	        }
	    }
	}

其中，泛型T即为View层的实际类型。在未发现空指针异常之前，在Presenter层的实际类中是按照如下方式调用View层方法的：

	getViewLayer().doSomething()...

可以看出，`getViewLayer()`方法内部没有判空操作，在调用`getViewLayer()`方法之前也没有进行判空操作。这共同导致了空指针异常的发生。

既然没有做判空操作，这里就加上判空操作。但问题是：

1. 采用何种判空方式
2. 在哪里进行判空
3. 为空是如何处理

## 问题解决之Execption

使用异常捕获方式来处理空指针异常，可以避免app的崩溃，因此在调用`getViewLayer()`的方法块部分使用`try...catch`语句。如下：

	try {
    	//展示数据
        getViewLayer().showLocalFriendList(s);
     } catch (NullPointerException e) {
        e.printStackTrace();
     }

这样做最简单，但也最麻烦。因为Presenter的子类有多个，每个类中又调用了很多`getViewLayer()`方法，这样一个一个得加到什么时候。我就纳闷了，为什么编译器会强制要求我们处理JSON异常，但对空指针异常却不冒红（强制）？于是我就带着问题翻书，发现空指针是Runtime异常，而JSON异常是编译时异常。编译器只会强制要求我们捕获处理编译时异常...且继承自RuntimeException(包括)的所有子类异常都属于运行时异常，而不属于运行时异常的所有异常都归为编译时异常。

于是我就自定义了编译时异常，如下：

	public class ViewLayerNotFoundExecption extends Exception {
	    public ViewLayerNotFoundExecption() {
	    }
	
	    public ViewLayerNotFoundExecption(String detailMessage) {
	        super(detailMessage);
	    }
	}


现在的问题是：该如何将运行时异常转换为编译时异常呢？这样才能方便编译器提醒我把项目中所有的`getViewLayer()`方法都用`try...catch`包裹。

1. 采用何种判空方式：这里采用`if (mViewRef.get()!=null)`的方法
2. 在哪里进行判空：在`getViewLayer()`方法内部
3. 为空是如何处理：抛异常

代码如下：

	public abstract class BasePresenter<T> {
	    protected Reference<T> mViewRef;//View接口类型的弱引用
	
	    public void attachView(T view){
	        mViewRef = new WeakReference<T>(view);//建立关联
	    }
	
	    //获取View层
	    public T getViewLayer() throws ViewLayerNotFoundExecption {
	        if (mViewRef.get()!=null) {
	            return mViewRef.get();
	        }else{
	            throw new ViewLayerNotFoundExecption("view层为null");
	        }
	    }
	
	    //判断是否与View建立了关联
	    public boolean isViewLayerAvailable() {
	        return mViewRef!=null&&mViewRef.get()!=null;
	    }
	
	    //解除关联
	    public void detachView(){
	        if(mViewRef!=null){
	            mViewRef.clear();
	            mViewRef = null;
	        }
	    }
	}

这样，项目里就出现了大量的错误代码，我就可以一个一个的改了：

	try {
	    //展示数据
	    getViewLayer().showLocalFriendList(s);
	} catch (ViewLayerNotFoundExecption viewLayerNotFoundExecption) {
	    viewLayerNotFoundExecption.printStackTrace();
	}

操，这重复代码和无用功做的也太多了吧...没办法，于是我就忍者一个一个改吧...改完了，下班了，回家了，第二天继续上班...

## 问题解决之回调

若View层为空，就不要再调View层的方法了，也就不会产生空指针了。这是问题解决的关键。使用异常方式只能保证空指针发生的时候app不会崩溃而已。于是我就继续改进：

1. 采用何种判空方式：调用现成的`isViewLayerAvailable()`方法进行判空
2. 在哪里进行判空：在调用`getViewLayer()`方法（本质是获取View层引用）之前
3. 为空是如何处理：不再调用View层的方法

这样做减少了一个自定义的异常类，但总不能每次在调用`getViewLayer()`方法（本质是获取View层引用）之前都调用`isViewLayerAvailable()`方法进行判空吧，那得增加多少`if语句`。

`getViewLayer()`方法有返回值，没法在`getViewLayer()`内部修改，要么再写个方法？于是定义了一个以回调接口对象为参数的方法，也就增加了一个回调接口。

在定义这个回调接口的时候，刚开始我是以外部类（对应于内部类）的方式定义接口类型的回调的，但发现回调方法需要一个View层参数，泛型的继承又不知道该继承自什么（定义公共View层基类不合适）...在Presenter基类中不是有线程的View层泛型么，因此有了如下代码：

	public abstract class BasePresenter<T> {
	    protected Reference<T> mViewRef;//View接口类型的弱引用
	
	    public void attachView(T view){
	        mViewRef = new WeakReference<T>(view);//建立关联
	    }
	
	    //获取View层
	    public T getViewLayer() {
	            return mViewRef.get();
	    }
	
	    //判断是否与View建立了关联
	    public boolean isViewLayerAvailable() {
	        return mViewRef!=null&&mViewRef.get()!=null;
	    }
	
	    protected void viewLayerDo(ViewLayerCallBack<T> callBack) {
	        if (isViewLayerAvailable()) {
	            callBack.doSomething(mViewRef.get());
	        }
	    }
	
	    //解除关联
	    public void detachView(){
	        if(mViewRef!=null){
	            mViewRef.clear();
	            mViewRef = null;
	        }
	    }
	}

在Presenter子类中按照以下方式调用View层的方法：

	viewLayerDo(new ViewLayerCallBack<FriendsContract.View>() {
	    @Override
	    public void doSomething(FriendsContract.View view) {
	       view.showLocalFriendList(s);
	    }
	});

哟西，这才是问题解决的正确姿势啊。即减少了多于的重复`try...catch`语句，且在解决问题的根本逻辑上也说的过去：能不发生异常就不要发生异常，而非异常发生时该如何处理。前者是防患于未然（不让发生火灾），后者是解决问题要趁早（消防灭火）。

好了，写到这了，我要删除那些多于的`try...catch`语句了，谁让我有强迫症呢...