---
layout: post
title:  "Java 基础"
date:   2017-02-05
categories: 知识点
tags: Java
---

* content
{:toc}

本文主要记录一些零碎的Java基础知识。




## 内存之栈和堆

[Java堆内存的10个要点](http://blog.jobbole.com/13373/)

[详解Java的堆内存与栈内存的存储机制](http://www.jb51.net/article/77361.htm)

## final关键字

final修饰类：该类不能被继承
final修饰方法：该方法不能被重写
final修饰变量：该变量不能被二次赋值（只能被赋值一次）

final修饰局部变量

1. 基本类型：基本类型的值不能发生改变。
2. 引用类型：引用类型的地址值不能发生改变，但是该对象的堆内存的值是可以改变的（即对象内部的非final变量是可以改变的）。

final修饰成员变量的初始化时机

1. 被final修饰的变量只能赋值一次
2. 在构造方法完毕前，包括变量赋值代码块（针对非静态的常量）

