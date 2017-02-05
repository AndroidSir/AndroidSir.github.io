---
layout: post
title:  "View Animation(视图动画)"
date:   2017-02-05
categories: 知识点
tags: Android Animation 视图动画
---

* content
{:toc}

视图动画也称为补间动画，仅能对视图对象执行补间动画。补间动画可以计算诸如起始点（位置）、大小、旋转角度和其他的方面。




## 正文

补间动画可以对一个视图对象的位置、大小、旋转和透明度等内容进行一系列变换。假如有一个TextView对象，我们可以通过补间动画移动、旋转、使增大、使收缩这个TextVew。如果它还有背景图片，背景图片也将会变换。**animation**包下提供了所有用于补间动画的类。

通过xml文件或者java代码，我们可以定义一系列补间动画的信息。通过xml布局文件定义的补间动画比Java代码写的补间动画具有更好的可读性、可复用性、可更改性。以下的例子我们将使用xml做示范。

为了能够更加熟练的使用代码定义补间动画，引出了AnimationSet类和其他Animation子类。

动画指令可以定义我们预期的变化，比如什么时候开始，持续多长时间。变化可以依次也可以同时。比如我们可以让一个TextView从左移到右，然后旋转180度；或者从左移到右的过程中进行旋转。每一个特定的转换都需要一系列特定的参数（大小变换需要始末大小，旋转变换需要始末角度等），和一系列公用的参数（例如开始时间和起始时间）。为了让一些变换同时发生，给它们相同的起始时间；为了让它们依次执行，计算开始时间的话则要加上前一个动画的持续时间。

补间动画的xml文件需存放于`res/anim/`文件夹下。一个xml文件必须有且仅有一个根节点，只能在alpha、scale、translate、rotate、插值器interpolator、和set之中选。set节点包含上述其它节点，当然，也可以包含其它set节点。xml中定义的变换**默认的是同时执行**，为了让它们依次执行，必须定义`startOffset`属性。下面的例子同时对View对象进行旋转和缩放，代码如下：

    <set android:shareInterpolator="false">
        <scale
         android:interpolator="@android:anim/accelerate_decelerate_interpolator"
         android:fromXScale="1.0"
         android:toXScale="1.4"
         android:fromYScale="1.0"
         android:toYScale="0.6"
         android:pivotX="50%"
         android:pivotY="50%"
         android:fillAfter="false"
         android:duration="700" />
     <set android:interpolator="@android:anim/decelerate_interpolator">
       <scale
            android:fromXScale="1.4"
            android:toXScale="0.0"
            android:fromYScale="0.6"
            android:toYScale="0.0"
            android:pivotX="50%"
            android:pivotY="50%"
            android:startOffset="700"
            android:duration="400"
            android:fillBefore="false" />
       <rotate
            android:fromDegrees="0"
            android:toDegrees="-45"
            android:toYScale="0.0"
        	android:pivotX="50%"
        	android:pivotY="50%"
       	    android:startOffset="700"
       	    android:duration="400" />
     </set>
    </set>

像`pivotX`的属性可以指定是相对于对象本身或相对于父容器进行动画，一定要使用合适的数据格式（“50”代表50%相对于父容器，或“50%”代表相对自身为50%）。也可以通过使用插值器来决定动画如何进行。


假如在`res/anim`目录下保存了一个`hyperspace_jump.xml`的文件，下面的代码将引用到它，并且将其应用到一个ImageView对象上。

    ImageView spaceshipImage = (ImageView) findViewById(R.id.spaceshipImage);
    Animation hyperspaceJumpAnimation = AnimationUtils.loadAnimation(this, R.anim.hyperspace_jump);
    spaceshipImage.startAnimation(hyperspaceJumpAnimation);

作为对startAnimation()方法的替代，我们也可以通过`Animation.setStartTime()`来设置动画开始时间，然后通过`View.setAnimation()`方法将动画指派给View对象。

注意：无论我们的动画怎样的移动或缩放，View对象的边界不会自动的去适配动画。也就是说，动画依然会在它所属的View的边界内被绘制而不会被裁减。一旦超过View的父View边界的时候，动画将会被裁减。

## 参考

[官方文档](https://developer.android.google.cn/guide/topics/graphics/view-animation.html)