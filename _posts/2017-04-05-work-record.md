---
layout: post
title:  "日更"
date:   2017-04-05
categories: 工作记录
tags: 坚持
---

* content
{:toc}

本文档记录当天的主要工作内容。坚持工作日日更----坚持，是一种品质。




## 代码

当popupwindow展示出来的时候，设置除了Popupwindow之外整个屏幕的透明度。

	private void backgroundAlpha(float baAlpha){
		WindowManager.layoutparam lp = this.getWindow().getAttributes();
		lp.alpha = bgAlpha;//0.0-1.0
		this.getWindow().setAttributes(lp);
	}

## 逻辑

* 产品九宫格显示规则：

	1. 单行
	2. 单行
	3. 单行
	4. 每行两张、双行
	5. 3.2双行
	6. 3.3双行
	7. 3.3.1三行
	8. 3.3.2三行
	9. 3.3.3三行

* 友爱岛二维码扫描处理：扫码成功后，若是公开岛屿，进入到岛屿详情界面（与之前逻辑一样）；若是私密岛屿，需判断用户是否加入了该私密岛屿，若已经加入，进入到岛屿详情界面（与之前逻辑一样）；若没有加入该岛屿，进入岛屿申请界面（IslandDetailsForJoin）

