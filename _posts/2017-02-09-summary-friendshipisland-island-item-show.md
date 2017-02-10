---
layout: post
title:  "岛屿条目展示总结"
date:   2017-02-09
categories: 模块总结
tags: ListView ScrollView 
---

* content
{:toc}

在做友爱岛模块的时候，有列表展示的需求。




方案之一是使用scrollView包含线性布局，线性布局再包含若干子布局，以达到滑动的效果。其中要展示的条目布局比较简单，不需要包含这些条目的属性，比如“我创建的”，“我发表的”等等，

	<com.qcsd99.www.views.ObservableScrollView
     android:id="@+id/osv_observable"
     android:layout_width="match_parent"
     android:layout_height="wrap_content"
     android:background="#FFEFEFEF"
     android:scrollbars="none">
     <LinearLayout
         android:layout_width="match_parent"
         android:layout_height="match_parent"
         android:orientation="vertical">

         &lt;!&ndash; 精品岛屿推荐 &ndash;&gt;

         <RelativeLayout
             android:layout_width="match_parent"
             android:layout_height="40dp"
             android:background="#f0f0f0">

             <TextView
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:layout_alignParentLeft="true"
                 android:layout_marginLeft="13dp"
                 android:layout_marginTop="12dp"
                 android:text="精品岛屿推荐"
                 android:textColor="#646464"
                 android:textSize="16sp" />
             <TextView
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:layout_alignParentRight="true"
                 android:layout_marginRight="13dp"
                 android:layout_marginTop="14dp"
                 android:text="更多"
                 android:textColor="#646464"
                 android:textSize="14sp" />
         </RelativeLayout>

         <LinearLayout
             android:id="@+id/ll_good_island_recommend"
             android:layout_width="match_parent"
             android:layout_height="wrap_content"
             android:orientation="vertical">

             <include layout="@layout/item_my_island" />

             <include layout="@layout/item_my_island" />

             <include layout="@layout/item_my_island"/>

             <include layout="@layout/item_my_island" />

         </LinearLayout>

         &lt;!&ndash; 优秀新开发岛屿推荐 &ndash;&gt;

         <RelativeLayout
             android:layout_width="match_parent"
             android:layout_height="40dp"
             android:background="#f0f0f0">

             <TextView
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:layout_alignParentLeft="true"
                 android:layout_marginLeft="13dp"
                 android:layout_marginTop="12dp"
                 android:text="优秀新开发岛屿推荐"
                 android:textColor="#646464"
                 android:textSize="16sp" />
             <TextView
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:layout_alignParentRight="true"
                 android:layout_marginRight="13dp"
                 android:layout_marginTop="14dp"
                 android:text="更多"
                 android:textColor="#646464"
                 android:textSize="14sp" />
         </RelativeLayout>
         &lt;!&ndash; listview &ndash;&gt;

         <LinearLayout
             android:id="@+id/ll_new_island_recommend"
             android:layout_width="match_parent"
             android:layout_height="wrap_content"
             android:orientation="vertical">

             <include layout="@layout/item_my_island" />

             <include layout="@layout/item_my_island" />

             <include layout="@layout/item_my_island" />
             <include layout="@layout/item_my_island" />
             <include layout="@layout/item_my_island" />
         </LinearLayout>
     </LinearLayout>
 	</com.qcsd99.www.views.ObservableScrollView>





方案之二是使用ListView进行列表展示，因为展示条目的时候还需要展示条目的属性，比如“我创建的”，“我发表的”等，因此在适配器中就需要做一些工作：

首先，传给适配器的集合的元素应该还是集合，子集合中分别放着不同属性的条目的数据；

接着，适配器中使用到的布局文件应该包含条目属性以及条目本身；

最后，在重写getView方法的时候，需要进行position与子集合元素个数的比较，判断当前应该展示那个子集合的数据，以及是否展示条目属性的布局。

适配器中的主要代码入下：

    
    public class SuggestOfFriendshipAdapter extends BaseAdapter {
    private int oldPosition;
    private Context context;
    private List<List<String>> list;


    public SuggestOfFriendshipAdapter(Context context, List<List<String>> list) {
        super();
        this.context = context;
        this.list = list;
    }

    @Override
    public int getCount() {
        return list.get(0).size() + list.get(1).size();
    }

    @Override
    public Object getItem(int arg0) {
        return arg0;
    }

    @Override
    public long getItemId(int arg0) {
        return 0;
    }

    @Override
    public View getView(int position, View convertview, ViewGroup arg2) {

        ViewHolder holder;

        if (convertview == null) {
            convertview = LayoutInflater.from(context).inflate(
                    R.layout.item_suggest_island, null);
            holder = new ViewHolder(convertview, context);
            convertview.setTag(holder);
        } else {
            holder = (ViewHolder) convertview.getTag();
        }
        if (position >= list.get(0).size()) {//如果当前位置大于他创建的岛的个数，就该显示优秀新开发岛屿推荐
            holder.tvIslandProperties.setText("优秀新开发岛屿推荐");
            if (position == list.get(0).size()) {
                holder.rlTitles.setVisibility(View.VISIBLE);
            } else {
                holder.rlTitles.setVisibility(View.GONE);
            }
            //显示的是加入的岛的信息
            holder.tvIslandName.setText(list.get(1).get(position - list.get(0).size()));
        } else {//先展示精品岛屿推荐
            holder.tvIslandProperties.setText("精品岛屿推荐");
            if (position == 0) {
                holder.rlTitles.setVisibility(View.VISIBLE);
            } else {
                holder.rlTitles.setVisibility(View.GONE);
            }
            //显示的是他创建的岛的信息
            holder.tvIslandName.setText(list.get(0).get(position));
        }
        holder.ivIslandPortrait.setBackgroundResource(R.drawable.homerightfragment2);
        return convertview;
    }

    class ViewHolder {
        /**
         * 岛屿性质：精品岛屿推荐，优秀岛屿推荐
         */
        @InjectView(R.id.tv_island_properties)
        TextView tvIslandProperties;
        /**
         * 岛屿名称
         */
        @InjectView(R.id.tv_island_name)
        TextView tvIslandName;
        /**
         * 岛屿头像
         */
        @InjectView(R.id.iv_island_portrait)
        ImageView ivIslandPortrait;
        Context mContext;
        /**
         * 标题头
         */
        @InjectView(R.id.rl_titles)
        RelativeLayout rlTitles;
        /**
         * 更多
         */
        @InjectView(R.id.tv_more)
        TextView tvMore;
        public ViewHolder(View view, Context context) {
            ButterKnife.inject(this, view);
            mContext = context;
        }
    }

getView方法填充的布局代码如下：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <RelativeLayout
        android:id="@+id/rl_titles"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="#f0f0f0">

        <TextView
            android:id="@+id/tv_island_properties"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_marginLeft="13dp"
            android:layout_marginTop="12dp"
            android:text="精品岛屿推荐"
            android:textColor="#646464"
            android:textSize="16sp" />

        <TextView
            android:id="@+id/tv_more"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:layout_marginRight="13dp"
            android:layout_marginTop="14dp"
            android:text="更多"
            android:textColor="#646464"
            android:textSize="14sp" />
    </RelativeLayout>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#ffffff"

        >
        <!--头像-->
        <ImageView
            android:id="@+id/iv_island_portrait"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_centerVertical="true"
            android:layout_marginBottom="5dp"
            android:layout_marginLeft="8dp"
            android:layout_marginTop="5dp"
            android:background="@drawable/myislandoffriendship1" />
        <!--岛屿名称-->
        <TextView

            android:id="@+id/tv_island_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="13dp"
            android:layout_marginTop="7dp"
            android:layout_toRightOf="@id/iv_island_portrait"
            android:text="逗比合集"
            android:textColor="#323232"
            android:textSize="18sp" />

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignLeft="@id/tv_island_name"
            android:layout_below="@id/tv_island_name"
            android:layout_marginTop="3dp"
            android:orientation="horizontal">
            <!--已产生-->
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="已产生"
                android:textColor="#909090" />
            <!--条数-->
            <TextView
                android:id="@+id/tv_how_many_new_content"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="123"
                android:textColor="#07b226" />
            <!--条新内容-->
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="条新内容"
                android:textColor="#909090" />
        </LinearLayout>


    </RelativeLayout>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="1px"
        android:background="#d6d7dc" />

	</LinearLayout>

另外一个需求就是，在下拉的时候，屏幕顶端要展示一个搜索框，这就需要监听到列表的滑动。

使用方案一（scrollView)的时候需要以下几步：

自定义接口IScrollViewListener；

自定义View继承自ScrollView，该View应具有自定义接口成员变量，应具有公共方法为该变量赋值，重写onScrollChanged方法，在其中调用接口变量的方法；

在activity或者fragment中实现该接口，并调用方法为自定义View的接口变量赋值。

自定义接口代码如下：

	public interface IScrollViewListener {
    	void onScrollChanged(ObservableScrollView scrollView,int x,int y,int oldx,int oldy);
	}

自定义View代码如下：

	public class ObservableScrollView extends ScrollView{
	//自定义接口变量
    private IScrollViewListener scrollViewListener = null;

    public ObservableScrollView(Context context) {
        super(context);
    }

    public ObservableScrollView(Context context, AttributeSet attrs,
                                int defStyle) {
        super(context, attrs, defStyle);
    }

    public ObservableScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
	//自定义公开方法为接口变量赋值
    public void setScrollViewListener(IScrollViewListener scrollViewListener) {
        this.scrollViewListener = scrollViewListener;
    }
	//重写onScrollChanged方法，在其中调用接口变量的方法
    @Override
    protected void onScrollChanged(int x, int y, int oldx, int oldy) {
        super.onScrollChanged(x, y, oldx, oldy);
        if (scrollViewListener != null) {
            scrollViewListener.onScrollChanged(this, x, y, oldx, oldy);
        }
    }

	}

在fragment中的方法如下：

	/*osvObservable.setScrollViewListener(new IScrollViewListener() {
    @Override
    public void onScrollChanged(ObservableScrollView scrollView, int x, int y, int oldx, int oldy) {
        Log.e(TAG, "oldx:" + oldx + "-------oldy:" + oldy + "-------x:" + x + "-------y:" + y);
        if (y < oldy) {
            llSearch.setVisibility(View.VISIBLE);
            Log.e(TAG, "下拉了");
            isShowSearch = true;
        }else if(y > oldy){//上拉
            llSearch.setVisibility(View.GONE);
            isShowSearch=  false;
        }
    }
	});
	osvObservable.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View view, MotionEvent motionEvent) {
        return false;
    }
	});*/

使用方案二（ListView)监听下拉的时候需要以下几步：

	//填充搜索栏布局
	searchView = inflater.inflate(R.layout.search_of_recommend_island_of_friendshipisland,null);
	//因为View.GONE直接作用在searchView上的时候和View.INVISIBLE的效果一样，所以要作用在次根布局
	llSearchContainer = searchView.findViewById(R.id.ll_search_container);
	//添加头部
	lvSuggest.addHeaderView(searchView);
	//次根布局设置不可见
	llSearchContainer.setVisibility(View.GONE);
	lvSuggest.setOnTouchListener(new View.OnTouchListener() {
    //成员变量
    float lastX, lastY;
    //按下的时候调用的方法
    private void motionActionDownEvent(float x, float y) {
        lastX = x;
        lastY = y;
    }
    //移动的时候调用的方法
    private void motionActionMoveEvent(float x, float y) {
        int dx = (int) (x - lastX);
        int dy = (int) (y - lastY);
        if (dy > 0) {
            //向下拉（向上看）的时候，添加的头部可见
            if(lvSuggest.getFirstVisiblePosition()==0){
                llSearchContainer.setVisibility(View.VISIBLE);
            }

        } else if (dy < 0) {
            //向上拉（向下看）的时候，添加的头部不可见
            if(lvSuggest.getFirstVisiblePosition()==1){
                llSearchContainer.setVisibility(View.GONE);
            }
        }
    }
    @Override
    public boolean onTouch(View view, MotionEvent motionEvent) {
        //获得动作类型
        final int action = motionEvent.getAction();
        //x,y是相对于该按钮左上点（控件本身）的相对位置而rawx,rawy始终是相对于屏幕的位置。
        //这里获得触摸点相对于屏幕左上角为坐标原点的x  y的值
        float x = motionEvent.getRawX();
        float y = motionEvent.getRawY();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                //按下
                motionActionDownEvent(x, y);
                break;
            case MotionEvent.ACTION_MOVE:
                //移动
                motionActionMoveEvent(x, y);
                break;
            default:
                break;
        }
        return false;
    }
	});
	lvSuggest.setOnScrollListener(new AbsListView.OnScrollListener() {
    @Override
    public void onScrollStateChanged(AbsListView absListView, int i) {
        switch (i) {
            case AbsListView.OnScrollListener.SCROLL_STATE_FLING:
                //开始滚动
                break;
            case AbsListView.OnScrollListener.SCROLL_STATE_IDLE:
                //停止
                break;
            case AbsListView.OnScrollListener.SCROLL_STATE_TOUCH_SCROLL:
                //正在滚动
                if(lvSuggest.getFirstVisiblePosition()==1){
                    llSearchContainer.setVisibility(View.GONE);
                }
                break;
        }
    }

    @Override
    public void onScroll(AbsListView absListView, int i, int i1, int i2) {

    }
	});
	lvSuggest.setAdapter(new SuggestOfFriendshipAdapter(getActivity(), list));
