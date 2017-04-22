---
layout: post
title:  "搜索框获焦，阴影层展示"
date:   2017-04-22
categories: 手刃需求
tags: 搜索框 阴影
---

* content
{:toc}

在岛屿推荐列表顶部，有一个搜索框。当用户点击搜索框的时候，iOS会出现灰层遮挡数据，从而突出搜索框。这里记录一下我的实现代码




## XML布局

	<RelativeLayout
        android:background="@drawable/shape_edit"
        android:focusable="true"                        //重点
        android:layout_width="match_parent"
        android:layout_height="30dp"
        android:layout_centerVertical="true"
        android:layout_marginBottom="5dp"
        android:layout_marginLeft="5dp"
        android:layout_marginRight="5dp"
        android:layout_marginTop="5dp"
        android:focusableInTouchMode="true">            //重点

        <EditText
            android:id="@+id/et_key_words"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_centerVertical="true"
            android:layout_marginLeft="4dp"
            android:background="@null"
            android:hint="搜索"
            android:paddingBottom="5dp"
            android:paddingLeft="5dp"
            android:paddingTop="5dp"
            android:singleLine="true"
            android:textSize="13sp"/>

        <ImageView
            android:id="@+id/iv_search"
            android:layout_width="21dp"
            android:layout_height="21dp"
            android:layout_alignParentRight="true"
            android:layout_centerVertical="true"
            android:layout_marginRight="8dp"
            android:src="@drawable/usedmarket150"/>
    </RelativeLayout>

## Java代码

灰层监听代码如下：

	@OnClick(R.id.v_grey)
    void vGreyListener(View view) {
        vGrey.setVisibility(View.GONE);
        etKeyWords.setFocusable(false);
        imm.hideSoftInputFromWindow(vGrey.getWindowToken(), 0);
    }

EditText监听如下：

	etKeyWords.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                vGrey.setVisibility(View.VISIBLE);
                etKeyWords.setFocusable(true);//设置输入框可聚集
                etKeyWords.setFocusableInTouchMode(true);//设置触摸聚焦
                etKeyWords.requestFocus();//请求焦点
                etKeyWords.findFocus();//获取焦点
                imm.showSoftInput(etKeyWords, InputMethodManager.SHOW_FORCED);// 显示输入法
            }
        });

    etKeyWords.setOnFocusChangeListener(new View.OnFocusChangeListener() {
            @Override
            public void onFocusChange(View v, boolean hasFocus) {
                if (hasFocus) {
                    vGrey.setVisibility(View.VISIBLE);                      //保证第一次点击编辑框，灰层展示
                }
            }
        });

	etKeyWords.setOnEditorActionListener(new TextView.OnEditorActionListener() {
            @Override
            public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
                if(actionId==EditorInfo.IME_ACTION_SEARCH){
                    vGrey.setVisibility(View.GONE);							//1
                    etKeyWords.setFocusable(false);							//2
                    imm.hideSoftInputFromWindow(vGrey.getWindowToken(), 0); //3
                    //搜索操作...
                    return true;
                }
                return false;
            }
        });

搜索图标的监听和键盘上搜索按钮的监听一样。当然，灰层的显示与隐藏可以通过动画的方法以渐变的形式实现。