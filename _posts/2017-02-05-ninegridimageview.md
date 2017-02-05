---
layout: post
title:  "开源控件--NineGridImageView--使用教程"
date:   2017-02-05
categories: 使用教程
tags: 使用教程 开源控件 
---

* content
{:toc}

本文介绍开源控件九宫格的使用方法。




## 正文

特性

- 设置图片之间的间隔

app:imgGap="4dp" 或nineGridImageView.setGap(int gap);

- 设置最大图片数：

app:maxSize="9" 或者 nineGridImageView.setMaxSize(int maxSize)

如果最大图片数小于等于0，则没有图片数的限制。

- 设置显示样式

app:showStyle="fill" 或 nineGridImageView.setShowStyle(int style);

默认样式是网格样式：STYLE_GRID；另外一种样式是：STYLE_FILL:

- 当只有一张图的时候，可以设置其显示大小，不让其显示的过小:

app:singleImgSize="120dp" 或 nineGridImageView.setSingleImgSize(int singleImgSize)

用法

1. 首先添加依赖

    `compile 'com.jaeger.ninegridimageview:library:1.0.1'`

2. 在布局文件中添加 NineGridImageView

	`<com.jaeger.ninegridimageview.NineGridImageView
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_height="wrap_content"
    android:layout_margin="16dp"
    android:layout_width="match_parent"
    app:imgGap="4dp"
    app:showStyle="fill"
    app:singleImgSize="120dp"/>`

3. 为 NineGridImageView 设置 NineGridImageViewAdapter

    `nineGridImageView.setAdapter(nineGridViewAdapter);`

下面是 NineGridImageViewAdapter.class 的源码:

	public abstract class NineGridImageViewAdapter<T> {

    protected abstract void onDisplayImage(Context context, ImageView imageView, T t);

    protected void onItemImageClick(Context context, int index, List<T> list) {
    
    }

    protected ImageView generateImageView(Context context) {
        GridImageView imageView = new GridImageView(context);
        imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
        return imageView;
    }
	}

T 是你图片的数据类型, 你可以简单的使用 String 类型也可以是你自定义的类型，在代码方式得到控件的时候可以指定泛型，用来确定图片的数据类型。

你必须重写 onDisplayImage(Context context, ImageView imageView, T t) 方法去设置显示图片的方式, 你可以使用 Picasso、Glide 、ImageLoader 或者其他的图片加载库，你也可以给 ImageView 设置一个占位图；

如果你需要处理图片的点击事件，你可以重写 onItemImageClick(Context context, int index, List<T> list) 方法，加上你自己的处理逻辑；

如果你要使用自定义的 ImageView，你可以重写 generateImageView(Context context) 方法， 去生成自定的 ImageView。

4. 给 NineGridImageView 设置图片数据：

    `nineGridImageView.setImagesData(List<T> imageDataList);`

可以知道，一个NineGridImageView中可能显示若干张图片，适配器中的`protected ImageView generateImageView(Context context)`方法返回的是每一张图片的类型，可以自定义，例如：

    @Override
    protected ImageView generateImageView(Context context) {
    NetworkImageView networkImageView = new NetworkImageView(context);
    return networkImageView;
    //return super.generateImageView(context);
    }

给NineGridImageView设置图片数据的方法中的参数的元素，最后是传递给适配器中`protected void onDisplayImage(Context context, ImageView imageView, Object o)`方法的第三个参数。到底显示多少张图片，是由`nineGridImageView.setImagesData(List<T> imageDataList);`方法中参数的元素个数决定的。这一点可以从NineGridImageView的源码看出。

在NineGridImageView的源码中，`public void setImagesData(List lists)`方法得到mImgDataList的值，而`private void layoutChildrenView()`方法正是通过mImgDataList的元素个数进行for循环来调用适配器的`protected void onDisplayImage(Context context, ImageView imageView, Object o)`方法。

## 参考

[开源控件：NineGridImageView](http://laobie.github.io/android/2016/03/06/nine-grid-iamge-view-libaray.html)

[九宫格展示图片使用教程](http://blog.csdn.net/fuxuemingzhu/article/details/50828935)