---
layout: post
title:  "Drawable Animation(帧动画)"
date:   2017-02-05
categories: 知识点
tags: AnimationDrawable 帧动画
---

* content
{:toc}

帧动画允许我们依次地加载一系列Drawable资源来创建一个动画。就像传统的动画思路：创建一系列不同的图片进行依次播放，像放电影。**AnimationDrawable**类是帧动画的基础。




### 正文

可以通过使用AnimationDrawable类的代码方式定义帧动画，但是在xml文件中列出不同的帧图画的方式则更为简单。帧动画的xml文件需要放置在`res/drawable`目录下。文件中需要说明每一帧图画的顺序以及持续时间。

xml文件能且仅能使用`<animation-list>`作为根节点，它还包含一些列`<item>`节点用于定义每一帧动画图片的信息：哪张图，以及持续时间。比如：

    <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    	android:oneshot="true">
    	<item android:drawable="@drawable/rocket_thrust1" android:duration="200" />
    	<item android:drawable="@drawable/rocket_thrust2" android:duration="200" />
    	<item android:drawable="@drawable/rocket_thrust3" android:duration="200" />
    </animation-list>

上述帧动画仅仅有三帧图片。通过设置`android:oneshot`属性值为true，可以使动画仅仅运行一次就停止，并维持在最后一帧的状态；若该属性的属性值为false的话动画就将循环播放。将上述代码命名为`rocket_thrust.xml`并保存在`res/drawable/`目录下，可以通过下面的代码引用并作为一个View对象的背景图并播放。代码如下：

    AnimationDrawable rocketAnimation;
    
    public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.main);
    
      ImageView rocketImage = (ImageView) findViewById(R.id.rocket_image);
      rocketImage.setBackgroundResource(R.drawable.rocket_thrust);
      rocketAnimation = (AnimationDrawable) rocketImage.getBackground();
    }
    
    public boolean onTouchEvent(MotionEvent event) {
      if (event.getAction() == MotionEvent.ACTION_DOWN) {
    	rocketAnimation.start();
    	return true;
      }
      return super.onTouchEvent(event);
    }

值的注意的是：`rocketAnimation.start()`方法没有在Activity的`onCreate()`方法中调用，是因为帧动画尚未完全依附于窗口。如果想要立即播放而非通过交互命令，我们可以在Activity的`onWindowFocusChanged()`方法中调用。 

## 参考

[官方文档](https://developer.android.google.cn/guide/topics/graphics/drawable-animation.html)