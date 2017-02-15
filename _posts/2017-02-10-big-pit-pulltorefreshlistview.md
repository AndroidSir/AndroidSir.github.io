---
layout: post
title:  "PullToRefreshListView使用心得"
date:   2017-02-10
categories: 手刃轮子
tags: PullToRefreshListView 
---

* content
{:toc}

帖子详情界面，采用PullToRefreshListView加两个HeaderView分别显示帖子信息和“点赞”、“评论”指示条，但是当帖子信息的长度超过一屏的时候，从下向上滑动的时候并非显示帖子的其余信息，而是显示“上拉加载数据......”。




## 问题1

当友爱岛需求变为用户可至少发布5000字的时候，帖子详情不能很好的展示这些数据，会出现摘要部分出现的问题。

1. 猜想可能是由于底部“评论及点赞”条的存在导致的该bug，但是将“评论及点赞”条去掉的时候，发现该bug仍然存在；
2. 打算换控件，使用`com.cpoopc.scrollablelayoutlib.ScrollableLayout`控件，和友爱岛个人信息界面使用同一个控件，该控件虽然可以显示过长（超过一屏）的信息，但是具体的评论及点赞列表却显示不出来。
3. 然后百度`pulltorefreshlistview headerview过长`，发现了[这里](http://bbs.csdn.net/topics/391863841)，因此判定可能是控件本身的bug，然后就有了以下代码，问题得到了解决。

		lvOfPostDetail.setMode(PullToRefreshBase.Mode.DISABLED);
        lvOfPostDetail.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                /*if(headerView2.getVisibility()==View.VISIBLE){
                    lvOfPostDetail.setMode(PullToRefreshBase.Mode.PULL_FROM_END);
                }else{
                    lvOfPostDetail.setMode(PullToRefreshBase.Mode.DISABLED);
                }*/
                if (lvOfPostDetail.getRefreshableView().getLastVisiblePosition()>=1){
                    lvOfPostDetail.setMode(PullToRefreshBase.Mode.PULL_FROM_END);
                }else{
                    lvOfPostDetail.setMode(PullToRefreshBase.Mode.DISABLED);
                }
                return false;
            }
        });