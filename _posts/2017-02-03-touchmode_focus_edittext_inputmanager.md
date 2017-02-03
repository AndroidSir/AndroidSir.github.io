---
layout: post
title:  "围绕EditText展开的知识点"
date:   2017-02-03
categories: Android
tags: EditText TouchMode InputManager
---

* content
{:toc}

该文针对TouchMode是什么，是否可获取焦点，请求焦点方法的重载，以及代码设置软键盘的显示与隐藏等知识点进行了总结




###View.setFocusableInTouchMode

大多数Android设备都是触摸屏的，但是实际上Android设备也支持键盘操作，允许通过键盘来完成导航，点击，输入等。

当用户通过键盘（或者轨迹球）操作的时候，有必要聚焦当前接受输入的UI元素，例如，高亮（聚焦）某个按钮，让用户知道当前正在操作的UI元素是哪个。

但是，当用户使用触摸屏与设备交互的时候，始终聚焦当前UI元素就没有必要了，而且很丑陋；用户点击哪个元素，哪个元素就是当前元素，无需高亮标识。并且，通过触摸屏与设备交互的时候，点击某个UI元素也不会导致该元素聚焦，此时的高亮效果是由Pressed状态来完成的。也就是说，在Touch Mode模式之下，UI元素是不会进入聚焦状态的，即使调用requestFocus也不会。

那么，Android是如何区分这两种情况的呢？

答案就是Touch Mode。当用户开始通过键盘与设备交互的时候，设备就退出Touch Mode模式；当用户开始通过触摸屏与设备交互的时候，设备就进入Touch Mode模式。可以通过调用View的isInTouchMode来判断设备当前是否处于Touch Mode模式。

但是，也有例外情况。有些UI元素，即使是在Touch Mode的状态之下，也需要获得焦点，典型的就是Edittext。那么，这种情况该如何处理呢？

答案就是做特殊处理。Android规定，某些元素，即使是在Touch Mode模式下，也可以获得焦点。调用View的setFocusableInTouchMode(true)可以使View在Touch Mode模式之下仍然可获得焦点（像Edittext就是在内部设置了这个属性），调用isFocusableInTouchMode可以判断View是否可在Touch Mode模式下聚焦。

***

当用户直接使用keys或trackball与UI进行交互的时候, 必须先使目标控件获取焦点(比如按钮)，这样用户才会注意到是什么控件接收输入. 然而如果设备支持触摸手势的话, 用户可能使用触摸屏与UI进行交互, 这个时候就没有必要将目标控件高亮显示了（即，获取焦点）. 因此就产生了这样一种交互模式叫"touch mode ."
对于一个拥有触摸屏功能的设备而言, 一旦用户用手点击屏幕, 设备立刻进入touch mode . 这时候被点击的控件只有isFocusableInTouchMode()方法返回true的时候才会 focusable , 比如EditText控件. 其他可以触摸的控件, 比如按钮, 当被点击的时候不会获取焦点; 它们只是简单地执行onClick事件而已.
任何时候只要用户点击key或滚动trackball, 设备就会退出touch mode ,并且找一个view将焦点置于其上. 此时用户可以不使用触摸手势了.
touch mode 在整个系统运行期间都是有效的(在任何activities中). 如果想要查询当前处于何种状态, 你可以调用View#isInTouchMode()来看看当前是否处于touch mode .

对于android触屏手机来说 在整个系统的运行期间都是处于TouchMode模式 只要是在该模式下正常的控件都不能够获取焦点  除非设置了isFocusableInTouchMode()  在android提供的控件中EditTest就属于在TouchMode模式下可以获取焦点的例外  

**setFocusable这个是用键盘是否能获得焦点**

	/**
     * Set whether this view can receive the focus.规定该view是否可以获取到焦点
     *
     * Setting this to false will also ensure that this view is not focusable
     * in touch mode.设置为false将保证该view在touch mode下是不能获取焦点的
     *
     * @param focusable If true, this view can receive the focus.
     *
     * @see #setFocusableInTouchMode(boolean)
     * @attr ref android.R.styleable#View_focusable
     */
    public void setFocusable(boolean focusable) {
        if (!focusable) {
            setFlags(0, FOCUSABLE_IN_TOUCH_MODE);
        }
        setFlags(focusable ? FOCUSABLE : NOT_FOCUSABLE, FOCUSABLE_MASK);
    }

**setFocusableInTouchMode这个是触摸是否能获得焦点**

	/**
     * Set whether this view can receive focus while in touch mode.规定该view在touch mode下是否可以获取焦点
     *
     * Setting this to true will also ensure that this view is focusable.设置为true将确保该view是在touch mode下可获得焦点的
     *
     * @param focusableInTouchMode If true, this view can receive the focus while
     *   in touch mode.
     *
     * @see #setFocusable(boolean)
     * @attr ref android.R.styleable#View_focusableInTouchMode
     */
    public void setFocusableInTouchMode(boolean focusableInTouchMode) {}

[TouchMode下的focus问题](http://blog.csdn.net/vincent_czz/article/details/6608781)
[资料2](http://www.fx114.net/qa-89-29940.aspx)

###View.requestFocus()

    public final boolean requestFocus ()
    
    Call this to try to give focus to a specific view or to one of its descendants.
	A view will not actually take focus if it is not focusable (isFocusable() returns false), or if it is focusable and it is not focusable in touch mode (isFocusableInTouchMode()) while the device is in touch mode. See also focusSearch(int), which is what you call to say that you have focus, and you want your parent to look for the next one. 
	This is equivalent to calling requestFocus(int, Rect) with arguments FOCUS_DOWN and null.

    Returns
    Whether this view or one of its descendants actually took focus.

	
[如何让一个控件能主动获取到焦点](http://blog.sina.com.cn/s/blog_62f987620100qwih.html)

[资料2](http://blog.csdn.net/cdl343794966/article/details/13167999)

[requestFocus 标签的用法,Edittext主动弹出软键盘](http://blog.csdn.net/ouyang_peng/article/details/46957281)

[requestFocus这个标签是干什么的?](http://www.imooc.com/qadetail/64495)


###InputMethodManager

[关于InputMethodManager的使用方法](https://my.oschina.net/jbcao/blog/61035)

[Android自动打开和关闭软键盘](http://blog.csdn.net/huiguixian/article/details/42103055)

获取方式：`((InputMethodManager) getSystemService(INPUT_METHOD_SERVICE))`

显示：使用该方法之前，view需要获得焦点，可以通过requestFocus()方法来设定。

	/**
     * Flag for {@link #showSoftInput} to indicate that this is an implicit
     * request to show the input window, not as the result of a direct request
     * by the user.  The window may not be shown in this case.
     * 表明是隐式请求显示软键盘，和用户直接请求结果不同。该窗口可能不会show。
     * 
     */
    public static final int SHOW_IMPLICIT = 0x0001;
    
    /**
     * Flag for {@link #showSoftInput} to indicate that the user has forced
     * the input method open (such as by long-pressing menu) so it should
     * not be closed until they explicitly do so.
     * 表明是用户强制打开软键盘，所以它不应该被关闭，直到他们明确这样做。
     */
    public static final int SHOW_FORCED = 0x0002;
    
    /**
     * Synonym（同义词） for {@link #showSoftInput(View, int, ResultReceiver)} without
     * a result receiver: explicitly request that the current input method's
     * soft input area be shown to the user, if needed.
     * 
     * @param view The currently focused view, which would like to receive
     * soft keyboard input.
     * @param flags Provides additional operating flags.  Currently may be
     * 0 or have the {@link #SHOW_IMPLICIT} bit set.
     */
    public boolean showSoftInput(View view, int flags) {
        return showSoftInput(view, flags, null);
    }

隐藏：

	/**
     * Flag for {@link #hideSoftInputFromWindow} to indicate that the soft
     * input window should only be hidden if it was not explicitly shown
     * by the user.
     * 表明如果不是用户明确要显示软键盘，则它应该隐藏。
     */
    public static final int HIDE_IMPLICIT_ONLY = 0x0001;
    
    /**
     * Flag for {@link #hideSoftInputFromWindow} to indicate that the soft
     * input window should normally be hidden, unless it was originally
     * shown with {@link #SHOW_FORCED}.
     * 表明软键盘通常应自动隐藏，除非被强制show。
     */
    public static final int HIDE_NOT_ALWAYS = 0x0002;

    /**
     * Synonym for {@link #hideSoftInputFromWindow(IBinder, int, ResultReceiver)}
     * without a result: request to hide the soft input window from the
     * context of the window that is currently accepting input.
     * 
     * @param windowToken The token of the window that is making the request,
     * as returned by {@link View#getWindowToken() View.getWindowToken()}.
     * @param flags Provides additional operating flags.  Currently may be
     * 0 or have the {@link #HIDE_IMPLICIT_ONLY} bit set.
     */
    public boolean hideSoftInputFromWindow(IBinder windowToken, int flags) {
        return hideSoftInputFromWindow(windowToken, flags, null);
    }