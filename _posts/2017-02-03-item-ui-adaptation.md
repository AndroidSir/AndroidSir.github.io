---
layout: post
title:  "Item适配问题"
date:   2017-02-03
categories: 手刃bug
tags: 适配  dp sp px
---

* content
{:toc}

Android布局的难点在不同机型的适配，在一个手机上显示的正常，但在另一个手机上就未必能显示的一模一样。这可能由于手机硬件（用户）的原因，也可能由于代码的原因。





需求：如下图所示的ListView的Item的布局

<div  align="center">    
<img src="http://a3.qpic.cn/psb?/V11DxkGh190yEc/7z*hM0gg2LQiXW9IXtuQJT3T5uIvJNQBTxx1m1O1Mh0!/b/dB8BAAAAAAAA&bo=gAJvBAAAAAADB8s!&rf=viewer_4" width = "800" height = "1500" />
</div>

看到这张效果图的时候，我的第一直觉是判断每个item的高度。由拉弓图可知：150+10+10=170，布局文件的root直接给了如下代码：

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/rl_foot"
    android:layout_width="match_parent"
    android:layout_height="170dp"
    android:background="#ffffff">

然后，

1. 确定“杨过”的位置，居左13、居上13
2. 确定描述的位置，和“杨过”居左对齐、上方向距“杨过”13dp，距右10（没标注，自己发挥）
3. 图片宽120，高150，位于parent的right，距右13，距上10（或者在parent的垂直居中）

难点：末尾的省略号如何显示？


**做法1**：使用TextView的如下属性

	android:ellipsize="end"
    android:maxLines="6"

或调用方法`tv.setEllipsize(TextUtils.TruncateAt.END)`，该方法的解释是：

    Causes words in the text that are longer than the view is wide to be ellipsized instead of broken in the middle. You may also want to setSingleLine or setHorizontallyScrolling to constrain the text to a single line. Use null to turn off ellipsizing. If setMaxLines has been used to set two or more lines, only TextUtils.TruncateAt.END and TextUtils.TruncateAt.MARQUEE are supported (other ellipsizing types will not do anything).

翻译为：倘若这个TextView要显示的文本长于TextView的宽度，调用该方法可以让显示效果为省略大小（ellip  size）而非直接从中间断开：即显示一个省略号。如果需要在一行（不换行）并水平滑动查看完整的内容，该方法可以传null值来关闭省略大小。如果设置的最大行数不小于两行，该方法只能传`TextUtils.TruncateAt.END `或`TextUtils.TruncateAt.MARQUEE`两个枚举类型的参数，其他类型的参数将无任何效果。

**效果1**：在华为Mate S`HUAWEI CRR-UL00,Android 6.0,(API 23)`上显示的结果为6行且有省略号，但在`Samsung GT-N7102, Android 4.3 , (API 18)`上为5行且没有省略号。这该怎么办？

于是我就**做法2**：在适配器中为tv设置过text后调用以下代码。

	int lineHeight = tv.getLineHeight();				//获取行高     A
	int height = tv.getHeight();						//获取tv的高度 C
	float lineSpacingExtra = tv.getLineSpacingExtra();  //获取行间距   B
														//行数设为X
	/**      因为XA+(X-1)B=C
	 *		所以有X=(C+B)/(A+B)
	 */
	double targetLines = (MyUtils.dip2px(ctx, 170 - 39 - 16) + lineSpacingExtra) / (lineHeight + lineSpacingExtra);
	tv.setMaxLines((int) targetLines);
	tv.setEllipsize(TextUtils.TruncateAt.END);

**效果2**：在两个手机上显示的都是4行且末尾有省略号。还不是预期的结果（都是6行且末尾有省略号）。

于是我就**做法3**：分别获取不同手机上展示描述内容的TextView的一些信息，结果如下：

华为：

	tv高px=346           行高px=60         行间距px=18.0
	tv高dp=115         行高dp=20       行间距dp=6

三星：

	tv高px=225           行高px=45         行间距px=12.0
	tv高dp=113         行高px=23       行间距px=6 

可以发现：

1. 行间距由于在布局文件中通过`android:lineSpacingExtra="6dp"`属性设置，所以相同。
2. 从第二行及第四行的结果发现，115不等于113的原因就是标题占用的dp不同。但不应该啊，因此就测试同为16sp的“杨过”在不同手机上的高度。结果为：

华为：

	56px	即19dp      设置值：16sp

三星：

	43px	即22dp      设置值：16sp

这结果不应该，因此就百度sp与px之间的转换关系。发现了**附录1**的工具类，及**附录2**，由附录2可知，Android系统允许用户自定义文字尺寸大小（小、正常、大、超大等等），于是就马上查看两个手机的字体大小。果不其然，三星字体是“正常”，华为字体是“小”。将华为的字体设置为“正常之后”，此时“杨过”的高度为22dp。

**效果3**：字体大小相同（正常）时，两个手机效果一样，但用户众多，不同的用户手机的字体大小定有差别，因此应继续优化。

**做法4**：转换思路，不能限制root的高，否则TextView的高度也就限制了，让root的高度为wrap，然后改变子View的布局方式。于是有了如下代码：
	
	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    android:id="@+id/rl_foot"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:background="#ffffff">
	
	    <TextView
	        android:id="@+id/tv_title"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_marginLeft="13dp"
	        android:layout_marginTop="13dp"
	        android:textColor="#323232"
	        android:textSize="16sp" />
	
	    <TextView
	        android:id="@+id/tv_desc"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_alignLeft="@id/tv_title"
	        android:layout_below="@id/tv_title"
	        android:layout_marginRight="10dp"
	        android:layout_marginTop="13dp"
	        android:paddingBottom="10dp"       //我有注释
	        android:layout_toLeftOf="@id/riv"
	        android:lineSpacingExtra="6dp"
	        android:ellipsize="end"
	        android:maxLines="6"
	        android:textColor="#909090"
	        android:textSize="14sp" />
	
	    <com.makeramen.roundedimageview.RoundedImageView
	        android:id="@+id/riv"
	        android:layout_width="120dp"
	        android:layout_height="match_parent"
	        android:layout_alignBottom="@id/tv_desc"
	        android:layout_alignTop="@id/tv_title"
	        android:layout_alignParentRight="true"
	        android:layout_marginRight="13dp"
	        android:layout_marginBottom="10dp"
	        android:scaleType="centerCrop"
	        app:riv_border_width="0dp"
	        app:riv_corner_radius="8dp" />
	</RelativeLayout>

注意有注释的一行，此行若使用`android:layout_marginBottom="10dp"`是无效的，tv_desc的底部与root的底部没有距离。

# 总结

1. 使用TextView使尽量不使其高度固定，否则可能显示垂直方向的半行问题
2. 内容太长显示省略号的时候，两个属性（方法）问题即可搞定：max...属性，ellipsize属性控制省略号显示的位置。

# 在解决该问题的整个过程中的发现：

1. 如果TextView调用`setText()`之后直接调用`getLineCount()`以获取该TextView的行数，返回的结果将是0。需要为该TextView设置如下监听，将`getLineCount()`方法的调用写在该监听里，且`onPreDraw()`方法默认返回false，应返回true，否则会循环执行该方法体：

	
		tvTitle.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
            @Override
            public boolean onPreDraw() {
                
                return true;
            }
        });
2. 如果TextView没有设置过`maxLines`这个属性，通过`getMaxLines()`方法获得的值将是一个很大的值：2147483647，即`Integer.MAX_VALUE`
3. 如果没有设置`maxLines`这个属性，TextView的高度非wrap，如果内容实际很长，但只显示了部分行，此时在`onPreDraw()`调用的`getLineCount()`方法获得的行数为假定完全显示的行数。即假设：有11行的内容，由于TextView的大小限制只能看到3行，通过方法获取到的行数为11



## 附录1

dp(dip): device independent pixels(设备独立像素). 不同设备有不同的显示效果,这个和设备硬件有关，一般我们为了支持WVGA、HVGA和QVGA 推荐使用这个，不依赖像素。
dp也就是dip，这个和sp基本类似。如果设置表示长度、高度等属性时可以使用dp 或sp。但如果设置字体，需要使用sp。

dp是与密度无关，sp除了与密度无关外，还与scale无关。如果屏幕密度为160，这时dp和sp和px是一 样的。1dp=1sp=1px，但如果使用px作单位，如果屏幕大小不变（假设还是3.2寸），而屏幕密度变成了320。那么原来TextView的宽度 设成160px，在密度为320的3.2寸屏幕里看要比在密度为160的3.2寸屏幕上看短了一半。但如果设置成160dp或160sp的话。系统会自动 将width属性值设置成320px的。也就是160 * 320 / 160。其中320 / 160可称为密度比例因子。也就是说，如果使用dp和sp，系统会根据屏幕密度的变化自动进行转换。

px: pixels(像素). 不同设备显示效果相同，一般我们HVGA代表320x480像素，这个用的比较多。
pt: point，是一个标准的长度单位，1pt＝1/72英寸，用于印刷业，非常简单易用；
sp: scaled pixels(放大像素). 主要用于字体显示best for textsize。



    /** 
     * dp、sp 转换为 px 的工具类 
     *  
     * @author fxsky 2012.11.12 
     * 
     */  
    public class DisplayUtil {  
        /** 
         * 将px值转换为dip或dp值，保证尺寸大小不变 
         *  
         * @param pxValue 
         * @param scale 
         *            （DisplayMetrics类中属性density） 
         * @return 
         */  
        public static int px2dip(Context context, float pxValue) {  
            final float scale = context.getResources().getDisplayMetrics().density;  
            return (int) (pxValue / scale + 0.5f);  
        }  
      
        /** 
         * 将dip或dp值转换为px值，保证尺寸大小不变 
         *  
         * @param dipValue 
         * @param scale 
         *            （DisplayMetrics类中属性density） 
         * @return 
         */  
        public static int dip2px(Context context, float dipValue) {  
            final float scale = context.getResources().getDisplayMetrics().density;  
            return (int) (dipValue * scale + 0.5f);  
        }  
      
        /** 
         * 将px值转换为sp值，保证文字大小不变 
         *  
         * @param pxValue 
         * @param fontScale 
         *            （DisplayMetrics类中属性scaledDensity） 
         * @return 
         */  
        public static int px2sp(Context context, float pxValue) {  
            final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;  
            return (int) (pxValue / fontScale + 0.5f);  
        }  
      
        /** 
         * 将sp值转换为px值，保证文字大小不变 
         *  
         * @param spValue 
         * @param fontScale 
         *            （DisplayMetrics类中属性scaledDensity） 
         * @return 
         */  
        public static int sp2px(Context context, float spValue) {  
            final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;  
            return (int) (spValue * fontScale + 0.5f);  
        }  
    }  


## 附录2

[原网页](https://zhidao.baidu.com/question/264201411035450085.html)

sp：
　　与缩放无关的抽象像素（Scale-independent Pixel）。sp和dp很类似但唯一的区别是，Android系统允许用户自定义文字尺寸大小（小、正常、大、超大等等），当文字尺寸是“正常”时1sp=1dp=0.00625英寸，而当文字尺寸是“大”或“超大”时，1sp>1dp=0.00625英寸。类似我们在windows里调整字体尺寸以后的效果——窗口大小不变，只有文字大小改变。


# Android ListView的HeaderView与FooterView的布局大小与动态显示问题

**注意**：HeaderView与FooterView添加至listview的时候，应在设置适配器之前进行操作。

**需求**：根据关键字查询岛屿的时候，如果没有相应的匹配结果，需要展示暂无数据及图片。

**实现方式1**：推荐岛屿的无数据层与搜索岛屿的无数据层相互独立。listview所在的布局的root为帧布局，在顶层放置无数据层，该层距顶部的距离为搜索框的高度。当无搜索结果的时候让该层显示，否则让该层消失。

Q1：虽然搜索无相应数据时无数据层展示了出来，但是当在无数据层上下滑动的时候搜索框依然会跟随手指进行移动，时隐时现。这种感觉很是不好。

A1：在代码中向无数据层添加触摸监听，消费发生在无数据层上的一切事件。

Q2：手指定位在搜索框上滑动的时候，搜索框依然会跟随手指进行移动，时隐时现。

A2：换思路...于是就有了实现方式2。


**实现方式2**：将搜索无结果的无数据层作为listVieww的footerview，进行动态的显示与隐藏。

Q1：footerview的布局文件中，写的root的高为match_parent，但是显示出来的时候仅有差不多一个item的高度，不能填充空白区域。

A1：代码中动态调整布局的高度，这时候就需要根据手机的宽度和高度与搜索框的高度的差值进行相应判断，合理赋值。

Q2：使用setVisibility（）方法对footerview进行动态展示与隐藏的时候，发现没有效果。

A2：问百度，发现了[这里](http://www.cnblogs.com/smyhvae/p/5810471.html)。

总结：

- HeaderView与FooterView的宽高最好在布局文件中就以数值确定下来，全屏需用代码方式设置。
- 使用setVisibility（）方法进行动态展示与隐藏的话受用对象应该为次root（HeaderView与FooterView皆是）。