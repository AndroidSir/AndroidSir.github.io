---
layout: post
title:  "小技巧之Popupwindow"
date:   2017-02-05
categories: 手刃需求
tags: Popupwindow
---

* content
{:toc}

+ 点击popupwindow如何让其消失？
+ 点击popupwindow的外部如何让其消失？




#关于popupwindow的一些方法

**问题1**：点击popupwindow外部区域让popupwindow消失，解决的方法有二：

- 调用方法
`popupWindow.setOutsideTouchable(true);`

- 设置`popupWindow.setFocusable(false);//focusable要为false(不设置默认的就是False)；`使其不可获取焦点，然后重写其所处的Activity的OnTouchEvent方法

代码如下：

    @Override
    public boolean onTouchEvent(MotionEvent event) {
	    // TODO Auto-generated method stub
	    if (popupWindow != null && popupWindow.isShowing()) {
		    popupWindow.dismiss();
		    popupWindow = null;
	    }
	    return super.onTouchEvent(event);
    }

    
**问题2**：点击PopupWindow自身时(非按钮控件时)使PopupWindow消失，解决的方法是：给PopupWindow的 contentView注册一个点击事件

	contentView.setOnClickListener(new View.OnClickListener() {
	    @Override
	    public void onClick(View v) {
		    if(popupWindow.isShowing()){
		    	popupWindow.dismiss();
		    }
	    }
    });



一些代码：

        //找控件
        tv = (TextView) findViewById(R.id.tv);
        //创建popupwindow
        popupWindow = new PopupWindow(this);
        //填充popupwindow布局
        View contentView = LayoutInflater.from(this).inflate(R.layout.layout_popupwindow, null);
        //设置popupwindow中的listView
        ListView lv_popupwindow = (ListView) contentView.findViewById(R.id.lv_popupwindow);
        lv_popupwindow.setAdapter(new ArrayAdapter<String>(this, R.layout.item_listview, items));
        //设置popupwindow的内容
        popupWindow.setContentView(contentView);
        //设置popupwindow的宽和高
        popupWindow.setWidth(300);
        popupWindow.setHeight(300);
        //设置popupwindow的背景
        popupWindow.setBackgroundDrawable(getResources().getDrawable(R.drawable.map));
        popupWindow.setOutsideTouchable(true);
        //为TextView设置监听
        tv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //设置popupwindow的位置
                popupWindow.showAsDropDown(v);
            }
        });

另一些代码：

    //创建popupwindow
    popupWindow = new PopupWindow(getActivity());
    //填充popupwindow布局
    View contentView = LayoutInflater.from(getActivity()).inflate(R.layout.popupwindow_what_is_car_know_number, null);
    //给popupwindow的ContentView设置监听，使其点击自身可以消失
    contentView.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
    if(popupWindow.isShowing()){
    popupWindow.dismiss();
    }
    }
    });
    //设置popupwindow的内容
    popupWindow.setContentView(contentView);
    //设置popupwindow的宽和高
    popupWindow.setWidth(ViewGroup.LayoutParams.MATCH_PARENT);
    popupWindow.setHeight(ViewGroup.LayoutParams.MATCH_PARENT);
    //设置popupwindow的背景
    popupWindow.setOutsideTouchable(true);
    popupWindow.showAsDropDown(v);