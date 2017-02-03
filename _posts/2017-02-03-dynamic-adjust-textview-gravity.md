---
layout: post
title:  "动态设置TextView的Gravity"
date:   2017-02-03
categories: Android
tags: 适配  TextView Gravity 
---

* content
{:toc}

需求：ListView中的item中有一个TextView，该TextView的宽度确定，根据要显示的内容长度动态调整文字的显示方式：不超过1行居中显示；超过1行的话无论第二行有几个字，左对齐显示。




看了鸿洋大神[打造的万能的ListView GridView适配器](http://blog.csdn.net/lmj623565791/article/details/38902805/)，也忍不住尝试了一把。

刚开始写的item布局文件如下：

	<?xml version="1.0" encoding="utf-8"?>
	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:id="@+id/fl_root"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content">
	
	    <RoundedImageView
	        android:id="@+id/iv__image"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:scaleType="centerCrop"
	        android:src="@drawable/friendshipislandbg4"
	        android:riv_border_width="0dp"
	        android:riv_corner_radius="8dp" />
	    <TextView
	        android:id="@+id/tv_name"
	        android:layout_width="match_parent"
	        android:layout_height="50dp"
	        android:layout_gravity="bottom"
	        android:background="@drawable/bs_school_name_bg"
	        android:gravity="center"
	        android:maxLines="2"
	        android:paddingLeft="20dp"
	        android:paddingRight="20dp"
	        android:textColor="#ffffff"
	        android:textSize="16sp" />
	
	</FrameLayout>

动态代码调整TextView的Gravity代码如下：

			@Override
            public void convert(ListviewViewHolder holder, TempDataBean item) {
                FrameLayout flRoot = (FrameLayout) holder.getView(R.id.fl_root);
				//	动态调整GridView中每一个item的大小，防止图片大小不同导致每一个item的大小不同
                ViewGroup.LayoutParams params = flRoot.getLayoutParams();
                params.height = displayWidth / 2 MyUtils.dip2px(ctx, 18);
                params.width = displayWidth / 2 MyUtils.dip2px(ctx, 18);
                flRoot.setLayoutParams(params);
                flRoot.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        startActivity(new Intent(BSItemActivity.this, BSDetailActivity.class));
                    }
                });
                //显示图片
                mImageLoader.displayImage(item.getSchoolImage(), (ImageView) holder.getView(R.id.iv_school_image), options, mImageLoadingListener);
                //用来判断是学校名字是一行或者两行
                holder.setText(R.id.tv_name, item.getName());
                TextView tv = (TextView) holder.getView(R.id.tv_name);
                int lineCount = tv.getLineCount();
				if(lineCount<=1){
					//	1行居中
					tv.setGravity(Gravity.CENTER);
				}else{
					//  2行居左
					tv.setGravity(Gravity.LEFT | Gravity.CENTER_VERTICAL);
				}

            }

编译，安装，跑起来，发现并不是预期的效果。现在的效果是第一次进入页面，未滑动，无论是一行还是两行都是居中显示。将第一屏滑出界面，并再次滑入界面的时候，1行的居中，2行的居左。我擦，这是什么情况。带着疑问，我将怀疑的目光转向了大神造的轮子。

	public static ListviewViewHolder get(Context context, View convertView, ViewGroup parent, int layoutId, int position) {
	        if (convertView == null) {
	            View itemView = LayoutInflater.from(context).inflate(layoutId, parent,false);
	            ListviewViewHolder holder = new ListviewViewHolder(context, itemView, parent, position);
	            holder.mLayoutId = layoutId;
	            return holder;
	        } else {
	            ListviewViewHolder holder = (ListviewViewHolder) convertView.getTag();
	            holder.mPosttion = position;
	            return holder;
	        }
	}

我怀疑这段代码是不是少了一些东西，大神没有考虑到可能会有动态修改布局的操作，而仅仅在`public void convert(ListviewViewHolder holder, TempDataBean item)`方法中考虑了数据替换的操作。

我又怀疑是不是使用`LayoutInflater`填充出来的View和复用的convertView有一些不同。难道应该在使用`LayoutInflater`进行填充View的时候就进行动态修改布局的操作，对复用的convertView修改布局参数莫非晚了？说干就干，我就写了一个抽象类，在使用`LayoutInflater`填充出View之后立即对View进行修改布局操作。抽象类如下：

	public abstract class SetLayoutProperty {
	    View itemView;
	    int position;
	
	    //获得填充出来的View及对应的位置
	    public void setItemView(View itemView, int position) {
	        this.itemView = itemView;
	        this.position = position;
	    }
	    
	    //动态改变布局参数的具体操作
	    public abstract void setLayoutProperty();
	
	    public View getItemView() {
	        return this.itemView;
	    }
	
	    public int getPosition() {
	        return position;
	    }
	}

然后修改了大神造的ViewHolder如下：

	public static ListviewViewHolder get(Context context, View convertView, ViewGroup parent, int layoutId, int position,SetLayoutProperty setLayoutProperty) {
	        if (convertView == null) {
	            View itemView = LayoutInflater.from(context).inflate(layoutId, parent,false);
				//主要增加了这个语句块
	            if (setLayoutProperty != null) {
	                setLayoutProperty.setItemView(itemView,position);
	                setLayoutProperty.setLayoutProperty();
	            }
	            ListviewViewHolder holder = new ListviewViewHolder(context, itemView, parent, position);
	            holder.mLayoutId = layoutId;
	            return holder;
	        } else {
	            ListviewViewHolder holder = (ListviewViewHolder) convertView.getTag();
	            holder.mPosttion = position;
	            return holder;
	        }
	}

由代码可以看出，如果抽象类不为空，说明具体操作实现了这个抽象类，调用`setLayoutProperty.setItemView(itemView,position)`方法将需要的内容传出，调用`setLayoutProperty.setLayoutProperty()`方法将执行具体的修改布局参数的逻辑。

又修改了大神造的Adapter如下：

	/**
     * 在复用之前（展示之前）对一些布局参数进行调整
     *
     * @return
     */
    public SetLayoutProperty setLayoutProperty() {
        return null;
    }

使用轮子的时候我选择这样写：

			@Override
            public SetLayoutProperty setLayoutProperty() {
                return new SetLayoutProperty() {
                    @Override
                    public void setLayoutProperty() {
                        //获取条目布局
                        View itemView = this.getItemView();
                        TextView tv = (TextView) itemView.findViewById(R.id.tv_school_name);
                        tv.setText(datas.get(getPosition()).getSchoolName());
                        int lineCount = tv.getLineCount();
                        LogSwitchUtils.logD(BSItemActivity.class,"学校名字的内容为："+datas.get(getPosition()).getSchoolName());
                        LogSwitchUtils.logD(BSItemActivity.class,"学校名字的行数为："+lineCount);
                        if (lineCount <= 1) {
                            tv.setGravity(Gravity.CENTER);
                        } else {
                            tv.setGravity(Gravity.LEFT | Gravity.CENTER_VERTICAL);
                        }
                    }
                };
            }

            @Override
            public void convert(ListviewViewHolder holder, TempDataBean item) {...}

效果是：无论是否滑出或者滑入，无论是1行还是2行，都是居中显示。通过打印的日志，我发现通过`getLineCount()`方法获得的数值都为0。不应该啊，让我看看这个方法，于是就发现：

	/**
     * Return the number of lines of text, or 0 if the internal Layout has not
     * been built.
     */
    public int getLineCount() {
        return mLayout != null ? mLayout.getLineCount() : 0;
    }

我了个去，此时此刻我只想知道如何让 `The internal Layout has been built.`或者什么时候`The internal Layout has been built.`

于是乎，我就找度娘，我给度娘说：`Return the number of lines of text, or 0 if the internal Layout has not been built.`然后度娘没有告诉我想要的答案：即如何让？或者什么时候？

我有接着问度娘：“getlinecount一直是0？”

度娘说了：TextView可以通过getLineCount获取行数，但是这里要在控件绘画后才能获取，否则调用这个函数返回值为0。但是我想在addText时就获得text的行数，这样我可以控制TextView的高度，请问有没有一些开源的代码或者想法可以实现在addText获取行数呢？

哈哈哈哈....在[这里](https://www.zhihu.com/question/19801466?sort=created)我知道了什么时候？

并且找到了方法

	final TextView totalTitleNo = (TextView) findViewById(R.id.tv_ac_sub_account);
	ViewTreeObserver vto = totalTitleNo.getViewTreeObserver();
	vto.addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
		@Override
		public boolean onPreDraw() {
			int lineCount = totalTitleNo.getLineCount();
			System.out.println(lineCount);
		}
	});

迫不及待，我赶紧试试：

			@Override
            public void convert(ListviewViewHolder holder, TempDataBean item) {
                FrameLayout flRoot = (FrameLayout) holder.getView(R.id.root_fl);
                ViewGroup.LayoutParams params = flRoot.getLayoutParams();
                params.height = displayWidth / 2 - MyUtils.dip2px(ctx, 18);
                params.width = displayWidth / 2 - MyUtils.dip2px(ctx, 18);
                flRoot.setLayoutParams(params);
                flRoot.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        startActivity(new Intent(BSItemActivity.this, BSDetailActivity.class));
                    }
                });
                //显示图片
                mImageLoader.displayImage(item.getSchoolImage(), (ImageView) holder.getView(R.id.iv_school_image), options, mImageLoadingListener);
                //为测试的控件设置值，用来判断是学校名字是一行或者两行
                holder.setText(R.id.tv_school_name, item.getSchoolName());
                final TextView tv = (TextView) holder.getView(R.id.tv_school_name);
                ViewTreeObserver vto = tv.getViewTreeObserver();
                //这个监听器看名字也知道了，就是在绘画完成之前调用的，在这里面可以获取到行数，当然也可以获取到宽高等信息。
                vto.addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
                    @Override
                    public boolean onPreDraw() {
                        int lineCount = tv.getLineCount();
                        LogSwitchUtils.logD(BSItemActivity.class, "学校名字的行数为：" + lineCount);
                        if (lineCount <= 1) {
                            tv.setGravity(Gravity.CENTER);
                            //tv.invalidate();
                        } else {
                            tv.setGravity(Gravity.LEFT | Gravity.CENTER_VERTICAL);
							//tv.invalidate();
                        }
                        return true;
                    }
                });
            }

一开始，我的最后一句代码是`return false；`但发现不行，该界面显示不出来，并且一直打Log，于是就返回了true。

一开始，最后一个注释其实并不是注释，但是当我将它注释掉，发现结果依然是正确的，它就成了注释。


在解决问题的时候，我还问过度娘：android Textview 一行居中，两行居左？

检索度娘得到了如下使用布局文件使TextView实现一行居中，两行居左的效果：

	<?xml version="1.0" encoding="utf-8"?>
	<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    android:id="@+id/root_fl"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content">
	
	    <com.makeramen.roundedimageview.RoundedImageView
	        android:id="@+id/iv_school_image"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:scaleType="centerCrop"
	        android:src="@drawable/friendshipislandbg4"
	        app:riv_border_width="0dp"
	        app:riv_corner_radius="8dp" />
	
		//该层使文字显示在底部
	    <RelativeLayout
	        android:layout_width="match_parent"
	        android:layout_height="50dp"
	        android:background="@drawable/bs_school_name_bg"
	        android:layout_gravity="bottom">
	
	        <LinearLayout
				//该层使单行居中
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:layout_centerInParent="true"
	            android:orientation="vertical">
	
	            <TextView
					//该层使两行居左
	                android:paddingLeft="20dp"
	                android:paddingRight="20dp"
	                android:id="@+id/tv_school_name"
	                android:layout_width="wrap_content"
	                android:layout_height="wrap_content"
	                android:gravity="left"
	                android:maxLines="2"
	                android:textColor="#ffffff"
	                android:textSize="16sp" />
	        </LinearLayout>
	    </RelativeLayout>
	
	</FrameLayout>

至此，解决一个问题我的所想，所做，所得都记录在了这里，便于以后复习。

## 结论：

1. 大神造的轮子没有错，是自己的理解有错。但敢于向权威质疑的精神是对的，在修改轮子的时候，顺便练习了使用抽象类和接口耦合高层模块（抽象）与底层模块（实现）的技术。
2. 实现TextView的单行居中，两行居左方法有：动态调整布局参数；完善布局文件。
3. 深刻体会了`tv.getLineCount()`方法。