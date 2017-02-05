---
layout: post
title:  "小技巧之Dialog的圆角"
date:   2017-02-05
categories: 小技巧
tags: Dialog 圆角
---

* content
{:toc}

在做罗曼蒂克“加入”界面的时候，如果用户没有点击“保存”，而直接返回的时候会弹出一个对话框（AlertDialog），通过自定义View，并为View中的部分控件设置监听即可完成功能逻辑。但是要求对话框的四个角分别是圆角，不管是自定义shape作为自定义View的background还是使用圆角图片，对话框的最外层始终是一个规规整整的矩形。为了解决这个bug，需要使用到自定义style。




## 正文

1.在资源文件中定义如下文件：

	<style name="dialog" parent="@android:style/Theme.Dialog">
    <item name="android:windowFrame">@null</item>
    <item name="android:windowIsFloating">true</item>
    <item name="android:windowIsTranslucent">true</item>
    <item name="android:windowNoTitle">true</item>
    <item name="android:background">@android:color/transparent</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:backgroundDimEnabled">true</item>
    <item name="android:backgroundDimAmount">0.6</item>
	</style>

2.引用资源文件：（重要）
引用的方法有两种：第一是通过自定义dialog实现，代码如下：

	public class RomanticExitAlertDialog extends Dialog {
    //一参的构造函数没有用到
    public RomanticExitAlertDialog(Context context) {
        super(context);
    }

    //两参的构造函数没有用到
    public RomanticExitAlertDialog(Context context, int themeResId) {
        super(context, themeResId);
    }
    //三参的构造没有用到
    protected RomanticExitAlertDialog(Context context, boolean cancelable, OnCancelListener cancelListener) {
        super(context, cancelable, cancelListener);
    }

	}

然后调用该类：

	exitAlertDialog = new RomanticExitAlertDialog(this,R.style.dialog);
	exitAlertDialog.setContentView(alertDialogView);

第二种方法是直接以style为参数构造builder，进而构造出alertdialog：

	exitAlertDialog =  new AlertDialog.Builder(this,R.style.dialog).create();

但这种方法有一些**要注意的地方**

必须先调用dialog的show方法之后才能调用dialog的setContentView方法，否则会出现运行时异常：

	requestFeature() must be called before adding content

eg：

	public boolean onKeyDown(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_BACK && event.getRepeatCount() == 0) {
        if(exitAlertDialog!=null){
            exitAlertDialog.show();
            exitAlertDialog.setContentView(alertDialogView);
        }
        return true;
    } else {
        return super.onKeyDown(keyCode, event);
    }
	}

## 参考

[requestFeature() must be called before adding content问题的解析](http://blog.csdn.net/u012422440/article/details/43888937)
