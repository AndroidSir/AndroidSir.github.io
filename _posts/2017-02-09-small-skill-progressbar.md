---
layout: post
title:  "小技巧之ProgressBar"
date:   2017-02-09
categories: 手刃需求
tags: ProgressBar 外观 监听
---

* content
{:toc}

ProgressBar指的是在某些操作过程中的可视指示器。通常向用户显示一个条形图，表示操作的进度。应用程序可以在实际进度增加时更改进度条的进度值。同时，一个进度条可以完成同时指示两个操作进度功能，比如视频播放过程中已缓冲的进度和当前看到的进度。

进度条也可以用于不确定事件的指示。在不确定模式下，进度条显示一个循环动画，没有进度指示。当任务的长度未知时，应用程序将使用此模式。不定的进度条可以是旋转轮，也可以是水平杆。




## 官方文档

下面的代码示例演示如何从工作线程中使用进度条来更新用户界面以通知用户进度：

	 public class MyActivity extends Activity {
	     private static final int PROGRESS = 0x1;
	
	     private ProgressBar mProgress;
	     private int mProgressStatus = 0;
	
	     private Handler mHandler = new Handler();
	
	     protected void onCreate(Bundle icicle) {
	         super.onCreate(icicle);
	
	         setContentView(R.layout.progressbar_activity);
	
	         mProgress = (ProgressBar) findViewById(R.id.progress_bar);
	
	         // Start lengthy operation in a background thread
	         new Thread(new Runnable() {
	             public void run() {
	                 while (mProgressStatus < 100) {
	                     mProgressStatus = doWork();
	
	                     // Update the progress bar
	                     mHandler.post(new Runnable() {
	                         public void run() {
	                             mProgress.setProgress(mProgressStatus);
	                         }
	                     });
	                 }
	             }
	         }).start();
	     }
	 }

默认情况下，进度条是一个旋转轮（依赖于手机系统版本的一个不确定的指标）。如果要换为水平进度条，应使用`widget.progressbar.horizontal`风格，代码如下：

	 <ProgressBar
	     style="@android:style/Widget.ProgressBar.Horizontal"
	     ... />

如果需要进度条显示真正的进度，则必须使用水平栏。然后可以调用`incrementprogressby()`或`setprogress()`增加进度。默认情况下，进度条在达到100时已满。如果有必要，可以使用`android：max`属性调整最大值（操作完成时候的值）。

另一个常见的进度条风格是`widget.progressbar.small`，显示了一个缩小版的旋转图形，用于等待内容加载。例如当要展示的内容需要从网络获取时，我们可以在默认布局的中心插入这种风格的进度条。刚开始的时候展示进度条，数据获取完毕的时候替换掉进度条。

Android提供的其他一些进度条风格如下：

	Widget.ProgressBar.Horizontal
	Widget.ProgressBar.Small
	Widget.ProgressBar.Large
	Widget.ProgressBar.Inverse
	Widget.ProgressBar.Small.Inverse
	Widget.ProgressBar.Large.Inverse

“Inverse”风格为相反的配色方案，如果应用程序使用一个浅色的主题（白色背景），则使用这种风格是必要的。

## 更改ProgressBar的外观

		/**
	     * created by zdd
	     * 将原生的ProgressBar展示的内容转换为奔跑的小人
	     * @param context
	     * @param pb
	     * @return
	     */
	    public static CanListenVisibleProgressbar changeProgressBarAppearance(Context context, CanListenVisibleProgressbar pb) {
	        ViewGroup.LayoutParams params = pb.getLayoutParams();
	        if (params != null) {
	            params.height = params.width = MyUtils.dip2px(context, 85);
	            pb.setLayoutParams(params);
	        }
	        pb.setBackground(context.getResources().getDrawable(R.drawable.progressdialogbackground));
	        pb.setIndeterminateDrawable(context.getResources().getDrawable(R.anim.frame3));
	        AnimationDrawable animationDrawable = (AnimationDrawable) pb.getIndeterminateDrawable();
	        animationDrawable.start();
	        return pb;
	    }

## 监听ProgressBar的显示

	/**
	 * Created by zdd on 2017/1/8 0008.
	 * 可监听是否可见的进度条
	 */
	public class CanListenVisibleProgressbar extends ProgressBar {
	    private Handler mListenersHandler;
	    private static final int DISMISS = 0x43;
	    private static final int SHOW = 0x45;
	    private OnDismissListener mOnDismissListener;
	    private OnShowListener mOnShowListener;
	 
	    public CanListenVisibleProgressbar(Context context) {
	        super(context);
	        if (mListenersHandler == null) {
	            mListenersHandler = new ListenersHandler();
	        }
	    }
	
	    public CanListenVisibleProgressbar(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        if (mListenersHandler == null) {
	            mListenersHandler = new ListenersHandler();
	        }
	    }
	
	    CanListenVisibleProgressbar(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        if (mListenersHandler == null) {
	            mListenersHandler = new ListenersHandler();
	        }
	    }
	
	    public void setOnDismissListener(final OnDismissListener listener) {
	        if (listener != null) {
	            mOnDismissListener = listener;
	        } else {
	            mOnDismissListener = null;
	        }
	    }
	
	    public void setOnShowListener(OnShowListener listener) {
	        if (listener != null) {
	            mOnShowListener = listener;
	        } else {
	            mOnShowListener = null;
	        }
	    }
	
	    public void show() {
	        mListenersHandler.sendEmptyMessage(SHOW);
	    }
	
	    public void dismiss() {
	        mListenersHandler.sendEmptyMessage(DISMISS);
	    }
	
	    private class ListenersHandler extends Handler {
	        @Override
	        public void handleMessage(Message msg) {
	            switch (msg.what) {
	                case DISMISS:
	                    setVisibility(INVISIBLE);
	                    if (mOnDismissListener != null) {
	                        mOnDismissListener.onDismiss();
	                    }
	                    break;
	                case SHOW:
	                    setVisibility(VISIBLE);
	                    if (mOnShowListener != null) {
	                        mOnShowListener.onShow();
	                    }
	                    break;
	            }
	        }
	    }
	
	    public interface OnDismissListener {
	        void onDismiss();
	    }
	
	    public interface OnShowListener {
	        void onShow();
	    }
	}

## 显示在标题上的进度条

还有一种进度条，可以直接在窗口标题上显示，这种金雕甚至不需要使用ProgressBar组件，它是直接由Activity的方法启用的。方法如下：

1. 调用Activity的`requestWindowFeature()`，根据需求传入`Window.FEATURE_INDETERMINATE_PROGRESS`显示不带进度的进度条或`Window.FEATURE_PROTRESS`显示带进度的进度条；
2. 调用Activity的`setProgressBarVisibility(boolean)`或`setProgressBarIndeterminateVisibility(boolean)`来控制进度条的显示和隐藏。

[参见疯狂Android讲义P116]()

## SeekBar（拖动条）的功能

拖动条（继承自进度条）和进度条非常相似，只是进度条采用颜色填充来表明进度完成的程度，而拖动条则通过滑块的位置来标识数值，且拖动条允许用户拖动滑块来改变值，因此拖动条通常用于对系统的某种数值进行调节。

[参见疯狂Android讲义P117]()

## RatingBar（星级评分条）的功能

星级评分条与拖动条有相同的父类：AbsSeekBar（继承自ProgressBar），因此它们十分相似。RatingBar与SeekBar的最大区别在于：RatingBar通过星星来表示进度（程度）。

[参见疯狂Android讲义P119]()