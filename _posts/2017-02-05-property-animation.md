---
layout: post
title:  "Property Animation(属性动画)"
date:   2017-02-05
categories: 知识点
tags: Animator 属性动画
---

* content
{:toc}

属性动画系统是一个允许我们对几乎任何对象进行的动画操作的健壮的框架。我们可以通过属性动画来改变任何对象的某一属性，即使这个对象不能够在屏幕上显示。属性动画在指定的时间内改变对象的一个（某些）属性值，而我们要做的是指定哪一个属性被“动画”，例如对象在屏幕上的位置，以及动画的持续时间，动画始末时刻的属性值。

 - 持续时间：属性动画默认的持续时间为300ms；
 - 时间插值器：指定在动画持续时间内，属性值以何种速率从起始值变化到默认值；
 - 重复次数和行为：指定单此动画执行完毕的时候是否继续正向重复执行，或者反向执行，以及动画重复的字数；
 - 动画集合：指定一个包含多个属性进行“动画”的动画组（Group）；
 - 帧刷新时间：指定动画执行过程中，帧与帧之间的刷新时间，默认值为10ms，但在真实应用程序中，它取决于系统是否繁忙以及系统的处理速度。




## 属性动画与视图动画（View Animation）的区别

视图动画又称为补间动画（**Tween**），视图动画系统仅仅具备为视图（View）对象提供动画的能力，它对非视图对象（non-View objects）无能为力。事实上，视图动画对于视图对象而言也只在很少的方面起作用，例如大小、旋转可行，但背景色就不起作用了。

视图动画的另一个缺陷就是它改变的只是视图对象的“外表”，而非视图对象本身。例如，通过视图动画让一个视图对象在屏幕上进行移动，移动前位置为A，移动后位置为B，即移动后与移动前“不在”同一个位置，但该视图对象的真实位置依然在A，B位置的内容只是系统绘出（drawn）的“虚假”图形，如果要操作这个视图对象，必须实现自己的操作逻辑。假如使用属性动画操作上述视图对象，它的位置就是真实的发生了变化。

属性动画克服了上述两个缺陷：无论对视图对象还是非视图对象都其作用；并且其作用范围涵盖颜色、位置、大小或者一切目标对象定义的任何属性。

然而，视图动画由于具备代码量更少，耗时较短的特点，当视图动画能够满足我们的需求的时候，不必使用属性动画。在不同的情景下选择更高效的动画系统，更能彰显出两种动画系统（属性与视图）存在的意义。
 
## 相关API

第一类：

**Animator**：该类提供了创建属性动画的的基本结构，通常情况下我们不会直接使用该类，因为它提供了很少的方法来支持操作动画属性值。大部分情况下我们使用的是其子类。

**ValueAnimator**：继承自Animator它是主要的在动画过程中，计算某一时刻属性值的计算引擎。它具备所有的核心功能方法，包括每一个动画的时序细节，动画是否重复的信息，监听到的更新事件，以及设置自定义属性值类型计算器的能力。属性动画的实现主要包括两方便的内容：一方面是计算不同时刻的属性值，另一方面是将计算出来的属性值赋值给目标对象。然而**ValueAnimator**是不能够实现第二方面的，因此必须自己监听属性值的更新并将其修改在目标对象上。

**ObjectAnimator**：继承自ValueAnimator，它允许我们在指定对象属性的时候同时指定目标对象，当计算出一个对应于某一时刻的新的属性值时，该类会自动更新目标对象的对应属性的属性值。大部分时间我们都在使用ObjectAnimator，因为它让新的属性值作用于目标对象的过程更为简单。但是，我们有些时候不得不直接使用ValueAnimator，因为ObjectAnimator有一些限制：要求目标对象必须具备一些特定的方法。

**AnimatorSet**：该类提供了多个属性动画“协调工作”的途径，不同属性动画之间可以“同步执行”，亦可以“依次执行”，或者“延时执行”。

第二类：

**TypeEvaluator**：接口，用于告诉系统将要改变的对象属性是什么类型的，以及如何计算要改变的属性。该接口的实现类依靠Animator提供的时序数据和属性始末值，来计算动画过程中各个时刻的属性值。系统主要提供了如下实现类。

**IntEvaluator**：是系统提供的默认的用于计算属性值为int类型的计算器。

**FloatEvaluator**：是系统提供的默认的用于计算属性值为float类型的计算器。

**ArgbEvaluator**：系统提供的默认的用于计算属性值为十六进制的色值类型的计算器。

当然，如果你要操作的目标对象的目标属性不在上述范围内，我们就需要实现TypeEvaluator接口来指定如何计算属性的动画值。除此之外，我们也可以通过自定义实现TypeEvaluator接口，为int，float，argb（color）类型的属性提供不同于系统默认的处理行为。

第三类：

**TimeInterpolator**：时间插值器（接口），用于指定在动画持续的过程中，属性值在单位时间内步进的增量大小，也就是属性值变化的速率，系统提供了如下实现类（以下列出的实现类只是常用的）。

**LinearInterpolator**：表明在动画持续过程中，属性值的变化速率是恒定的。

**AccelerateInterpolator**：表明在动画持续过程中，属性值的变化速率是一直增大的。

**DecelerateInterpolator**：表明在动画持续过程中，属性值的变化速率是一直减小的。

**AccelerateDecelerateInterpolator**：表明在动画持续过程中，属性值的变化速率是前半段一直增大，后半段持续减小的。

**CycleInterpolator**：为重复的属性动画指定了属性值的变化速率按照正弦曲线改变。

当然，如果系统提供的时间插值器不能够满足我们的需求，我们可以实现TimeInterpolator接口来构造时间插值。


## 通过ValueAnimator实现属性动画

我们可以通过调用ValueAnimator的诸如ofInt（），ofFloat（），ofObject（）等工厂方法来获取ValueAnimator对象。

比如：

    ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
    animation.setDuration(1000);
    animation.start();

当属性值类型非int，float的时候，可以进行如下操作：

    ValueAnimator animation = ValueAnimator.ofObject(new MyTypeEvaluator(), startPropertyValue, endPropertyValue);
    animation.setDuration(1000);
    animation.start();

上述两段代码中，并不能真实的作用于任何一个对象，因为ValueAnimator不能够直接操作对象或属性。想要解决这个问题，我们可以实现ValueAnimator定义的一个内部接口`ValueAnimator.AnimatorUpdateListener`来适时地处理诸如帧更新的事件，在实现这个接口的时候，我们可以通过调用ValueAnimator对象的getAnimatedValue()方法来获得为每一帧计算出来的属性值。

## 通过ObjectAnimator实现属性动画

ObjectAnimator整合了时序引擎和ValueAnimator的值计算能力，因此它可以完成对目标对象的指定属性的“属性动画 ”的操作。为了让对任何对象都能够更简单的实现属性动画，我们不必再实现ValueAnimator.AnimatorUpdateListener接口，因为指定的属性会自动更新。

初始化ObjectAnimator和ValueAnimator是相似的，不同的是初始化ObjectAnimator的时候，要指定目标对象和该对象的属性名称（String类型），以及动画始末该属性的属性值。例如：

    ObjectAnimator anim = ObjectAnimator.ofFloat(foo, "alpha", 0f, 1f);
    anim.setDuration(1000);
    anim.start();

为了确保指定属性在动画持续的过程中能够自动更新，必须注意以下几点：

 1. 目标对象的目标属性必须具有setter方法，如果该属性的setter方法不存在，可以：
  -  如果权限允许，增加setter方法；
  - 如果权限允许,写一个包裹类（wrapper），该包裹类应具有setter方法并指向原始目标对象的目标属性；
  - 使用ValueAnimator代替。
 2. 如果在对ObjectAnimator的工厂方法传实参的时候，values...的实参只传了一个值，该值将是动画结束的属性值。为了系统能够取到动画开始的属性值，目标对象的目标属性必须具备getter方法。
 3. 在指定属性动画的始末属性值的时候，始末属性值的类型必须和getter方法（如果该方法是必需的）和setter方法中参数的类型相一致，比如：`targetObject.setPropName(float) and targetObject.getPropName(float)`，当构造ObjectAnimator对象的时候，就必须`ObjectAnimator.ofFloat(targetObject, "propName", 1f)`
 4. 当操作的对象是视图对象的“可见属性”（颜色，大小）时，可能需要我们自己调用`invalidate()`强制使屏幕重绘对象本身。这步操作需要在`onAnimationUpdate()`回调方法中完成。视图对象的所有setter方法，例如`setAlpha()`和`setTranslationX()`都能够使视图对象实时的重绘，因此这些属性在设置新值得时候不必考虑重绘的问题。

## 使用AnimatorSet组织多个属性动画

在许多时候，一个动画的开始可能取决于另一个动画的开始或结束。Android系统允许使用AnimatorSet组织不同的属性动画，以便我们可以指定不同动画间的播放形式是同步、异步或是延时。当然，也可以将一个AnimatorSet对象嵌入另一个AnimatorSet对象中。

例如下面代码的实现效果就是依次：

 1. 播放bounceAnim动画；
 2. 同时播放squashAnim1,squashAnim2,stretchAnim1,stretchAnim2四个动画；
 3. 播放bounceBackAnim动画；
 4. 播放fadeAnim动画。

    AnimatorSet bouncer = new AnimatorSet();
    bouncer.play(bounceAnim).before(squashAnim1);
    bouncer.play(squashAnim1).with(squashAnim2);
    bouncer.play(squashAnim1).with(stretchAnim1);
    bouncer.play(squashAnim1).with(stretchAnim2);
    bouncer.play(bounceBackAnim).after(stretchAnim2);
    ValueAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
    fadeAnim.setDuration(250);
    AnimatorSet animatorSet = new AnimatorSet();
    animatorSet.play(bouncer).before(fadeAnim);
    animatorSet.start();

## Animation Listeners

我们可以在动画执行的过程中监听重要的事件。

**Animator.AnimatorListener**
 - onAnimationStart()，动画开始的时候调用；
 - onAnimationEnd()，动画结束的时候调用；
 - onAnimationRepeat()，动画重复的时候调用；
 - onAnimationCancel()，动画被取消的时候调用，被取消的动画也会调用onAnimationEnd()方法。

**ValueAnimator.AnimatorUpdateListener**
 - onAnimationUpdate()，动画的每一帧会调用。可以在监听这个事件的时候使用由ValueAnimator计算生成的属性值。使用ValueAnimator的时候可以调用getAnimatedValue()方法来获取新的属性值。

对于第一个接口的使用，我们可以继承AnimatorListenerAdapter类来代替实现Animator.AnimatorListener接口。AnimatorListenerAdapter类提供了所有需要实现的抽象方法的空实现。例如：

    ValueAnimatorAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
    fadeAnim.setDuration(250);
    fadeAnim.addListener(new AnimatorListenerAdapter() {
      public void onAnimationEnd(Animator animation) {
        balls.remove(((ObjectAnimator)animation).getTarget());
    }

## 通过LayoutTransition为布局的改变添加属性动画

和能够为视图对象提供简单的属性动画一样，属性动画系统也有能力为ViewGroup对象提供动画。

当在一个ViewGroup中的布局发生变化的时候，可以使用**LayoutTransition**类为其添加动画。这里所指的布局变化是指ViewGroup中的View对象隐藏和出现。隐藏可由两种情况导致：ViewGroup的removeView（），以及view的setVisibility（INVISIBLE/GONE）；出现可由两种情况导致：ViewGroup的addView（），以及view的setVisibility（VISIBLE）。从我们开始调用addView或者removeView，至控件到达它们应该到达的预期位置期间，我们可以定义这段时间的动画。

通过调用LayoutTransition对象的setAnimator()方法，传入一个Animator对象和不同的常量可以设置四种不同的动画：

 - 常量1：LayoutTransition.APPEARING  ViewGroup中有View出现的时候，View的动画；
 - 常量2：LayoutTransition.CHANGE_APPEARING ViewGroup中有View出现的时候，ViewGroup的动画；
 - 常量3：LayoutTransition.DISAPPEARING  ViewGroup中有View消失的时候，View的动画；
 - 常量4：LayoutTransition.CHANGE_DISAPPEARING  ViewGroup中有View消失的时候，ViewGroup的动画；

可以自定义我们自己的动画效果，方法如下：

    viewGroup.setLayoutTransition(mTransitioner);

当然也可以使用系统提供的默认动画效果,方法是：在布局文件中将目标ViewGroup的`android:animateLayoutchanges`属性值设置为`true`

## 使用接口TypeEvaluator

如果目标对象的目标属性的属性值类型是系统未提供的，你可以通过实现**TypeEvaluator**接口来定义自己的计算器（Evaluator）。系统提供的类型有int、float以及颜色值，分别对应IntEvaluator, FloatEvaluator和ArgbEvaluator。

实现该接口仅仅需要实现一个evaluate（）方法，该方法需要返回一个在动画过程中的某一时刻合适的目标属性值。例如FloatEvaluator的实现过程为：

    public class FloatEvaluator implements TypeEvaluator {
   		 public Object evaluate(float fraction, Object startValue, Object endValue) {
        		float startFloat = ((Number) startValue).floatValue();
        		return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);
    	 }
    }

注意：当ValueAnimator或ObjectAnimator运行的时候，它计算从动画开始，到当前时刻，已经过去的部分（介于0-1之间的数）；然后计算我们使用（传入）的插值器版本。插值器部分的内容就是`TypeEvaluator`中接收到的`fraction`参数，因此在计算目标属性某一个的属性值的时候不必再考虑插值器的影响。

## 使用插值器

一个插值器作为一个时间的函数，定义了如何影响动画过程中目标属性的属性值。例如，可以指定动画线性的发生，意味着在整个时间内均匀的移动卡给你；当然也可以使用非线性时间，比如开始加速，末尾减速。

动画系统中的插值器接收一个来自Animators（代表动画已经过去的时间）的分数。插值器修改这个分数，使结果一致于预期的目标。系统在`android.view.animation`包下提供了一系列公共的插值器，如果这些都不适合我们，可以实现TimeInterpolator接口来创造属于自己的插值器。

AccelerateDecelerateInterpolator的实现方式：

    public float getInterpolation(float input) {
    	return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
    }

LinearInterpolator的实现方式：

    public float getInterpolation(float input) {
    	return input;
    }

## 指定Keyframes(关键帧)

一个Keyframe对象包含一对“时间/值”对，这个“时间/值”对用于指定属性动画在特定时间的条件下的特定状态。每一个Keyframe可以有属于它本身的插值器，用来控制动画从上一个Keyframe的时间值到这一个Keyframe对象的时间值之间的动画的行为。

可以通过调用ofInt(), ofFloat(), 或ofObject()工厂方法来获得Keyframe对象；可以通过调用ofKeyframe()工厂方法来获得PropertyValuesHolder对象，一旦有了PropertyValuesHolder对象，就可以通过将其和目标对象作为参数获得动画对象，下面是示例代码：

    Keyframe kf0 = Keyframe.ofFloat(0f, 0f);
    Keyframe kf1 = Keyframe.ofFloat(.5f, 360f);
    Keyframe kf2 = Keyframe.ofFloat(1f, 0f);
    PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe("rotation", kf0, kf1, kf2);
    ObjectAnimator rotationAnim = ObjectAnimator.ofPropertyValuesHolder(target, pvhRotation)
    rotationAnim.setDuration(5000ms);


## 属性动画动画视图

属性动画系统优化了一些针对目标对象是视图对象的方法。我们知道，视图动画（补间动画）系统通过改变绘制视图方法的方法改造视图对象，这在每一个视图对象所在的容器（container：ViewGroup）中被处理，因为视图对象本身并没有属性可以被修改。结果是视图开始了动画，但视图对象本身并没有发生变化（移动的是幻影，本身没有动）。从Android 3.0开始，添加了新的属性和相应的setter、getter方法以避免上述漏洞。

属性动画可以通过改变真实的属性值在屏幕上操作视图对象。除此之外，视图对象党旗属性发生改变的时候可以自动调用`invalidate()`方法来刷新屏幕进行重绘。在View类中助于属性动画的新属性有：

 - translationX和translationY：距左，距上的平移；
 - rotation, rotationX, 和rotationY：2D（rotation）和3D上的旋转；
 - scaleX和scaleY：2D上的缩放；
 - pivotX和pivotY：参考位置，围绕该点进行缩放或旋转。默认是对象的中心；
 - x和y：简单实用的用来描述在容器中的最终位置的属性；
 - alpha：透明度

示例代码：

    ObjectAnimator.ofFloat(myView, "rotation", 0f, 360f);

## 通过ViewPropertyAnimator实现属性动画

**ViewPropertyAnimator**提供了一个简单的方法，仅用一个Animator对象来并行处理View对象的少量的属性。它的作用很像ObjectAnimator，因为它修改了视图对象的真实属性值，且更加高效。除此之外，使用ViewPropertyAnimator的代码更加简明易懂。下面是不同方法的对比，效果是同时修改视图对象的x和y属性。

多个ObjectAnimator对象：

    ObjectAnimator animX = ObjectAnimator.ofFloat(myView, "x", 50f);
    ObjectAnimator animY = ObjectAnimator.ofFloat(myView, "y", 100f);
    AnimatorSet animSetXY = new AnimatorSet();
    animSetXY.playTogether(animX, animY);
    animSetXY.start();

一个ObjectAnimator对象：

    PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", 50f);
    PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", 100f);
    ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvyY).start();

使用ViewPropertyAnimator：

    myView.animate().x(50f).y(100f);


## 在XML文件中声明属性动画

属性动画系统允许在xml文件中声明属性动画，在xml文件中定义属性动画，更以更加方便的在多个activity中复用，并可以更加简单的调整动画的顺序。

属性动画的xml文件应保存在`res/animator/`文件夹下，下面是声明属性动画类的的标签：

 - ValueAnimator： `<animator>`
 - ObjectAnimator：`<objectAnimator>`
 - AnimatorSet：`<set>`

示例代码如下：

    <set android:ordering="sequentially">
    <set>
        <objectAnimator
            android:propertyName="x"
            android:duration="500"
            android:valueTo="400"
            android:valueType="intType"/>
        <objectAnimator
            android:propertyName="y"
            android:duration="500"
            android:valueTo="300"
            android:valueType="intType"/>
    </set>
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueTo="1f"/>
    </set>

为了使用这个属性动画，必须使用AnimatorSet对象来填充xml资源，然后设置目标对象，实例代码如下：

    AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,R.anim.property_animator);
    set.setTarget(myObject);
    set.start();

## 参考

[官方文档](https://developer.android.google.cn/guide/topics/graphics/prop-animation.html)