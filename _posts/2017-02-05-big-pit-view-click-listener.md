---
layout: post
title:  "View的Listener的设置"
date:   2017-02-05
categories: 手刃bug
tags: Listener
---

* content
{:toc}

关于View设置监听需要注意的地方




## 概述：

在Android应用中，View类具有公有成员方法 `public void setOnClickListener(@Nullable OnClickListener l)`，因此，凡是继承自View的控件及容器都可以设置监听。

但在实际开发中，设置监听的对象必须为可操作的最小粒度控件。

设置监听的方法，需要一个接口类型的变量。根据实现接口的实体不同，可分为：

- 布局文件中对应控件配置onClick属性，此时应注意该布局文件必须作为activity中的setContentView方法的参数，否则会发生异常；

- 让承载需设置监听控件的activity或fragment实现相应接口，并重写相应方法。此时在重写相应方法的时候可以使用switch（v.getId（））方法。


- 定义接口类型的成员变量，此时重写方法的时候也可以使用switch（v.getId（））的类型；


- 自定义内部类实现指定接口，此时重写方法的时候也可以使用switch（v.getId（））的类型；


- 匿名内部类实现指定接口，由于是最小粒度控件调用设置监听的方法，因此此时参数的实现不能使用switch（v.getId（））的类型；

## 总结：

- 方法中的参数只要实现了相应接口即可，是谁并不重要；
- 参数必须大，但设置监听的控件必须为最小粒度，不能通过布局填充得到一个View之后，对View直接设置监听，这样是得不到View中的各个子控件的（监听得不到执行），即使是activity的setContentView方法也只是得到View，然后通过findViewById得到子控件之后，让子控件设置监听。
- 谁设置监听，switch（v.getId（））中的值才有可能是谁的。

## 延伸

为什么不能通过布局填充器填充得到一个View之后，对View直接设置监听，然后在监听里边通过switch语句为不同的id控件设置不同的操作？比如：

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        final Context context = this;
        View view = getLayoutInflater().inflate(R.layout.activity_main4, null);
        setContentView(view);
        Log.e("========",view.getClass().getSimpleName());
        view.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                switch (v.getId()) {
                    case R.id.btn_1:
                        Toast.makeText(context, "点击了按钮1", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.btn_2:
                        Toast.makeText(context, "点击了按钮2", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.btn_3:
                        Toast.makeText(context, "点击了按钮3", Toast.LENGTH_SHORT).show();
                        break;
                    case R.id.btn_4:
                        Toast.makeText(context, "点击了按钮4", Toast.LENGTH_SHORT).show();
                        break;
                }
            }
        });
    }

上述代码在点击四个按钮的时候都没有进行Toast。

通过Log可以看出，view的名字是Linearlayout，和布局文件中的root为Linearlayout对应。

代码之所以这样写：误认为设置监听的View是一个包含若干子View的非常复杂的ViewGroup，case的条件是对应子View的id。这犯了典型的形而上的错误，通过日志可以得到设置监听的仅仅是一个Linearlayout，无论在布局文件中root的id为多少（测试时没有设置），都不可能和任意一个case条件匹配，所以监听不会执行。

**即**：监听参数共用的时候需要switch（view's id）,设置监听的view只有一个id，不包含它含有的所有子view的id。