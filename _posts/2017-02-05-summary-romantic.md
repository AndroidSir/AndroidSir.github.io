---
layout: post
title:  "Summary之罗曼蒂克"
date:   2017-02-05
categories: Summary
tags: Summary
---

* content
{:toc}

罗曼蒂克主要功能分为三个界面：分别为RomanticHomeActivity、RomanticJoinActivity、RomanticSearchActivity，依次为主界面、加入界面、搜索范围界面。




## RomanticHomeActivity界面（主界面）：

<center>
<img src="http://a3.qpic.cn/psb?/V11DxkGh190yEc/ES4mXbT8pf8JQtus1VYF.gBr3PCgODE2ZdLCqQCYleg!/b/dHABAAAAAAAA&bo=gAJxBAAAAAAFB9M!&rf=viewer_4" height="50%" />
图 H-1. 罗曼蒂克主界面
</center>

**功能简介：**

该界面的主要功能是为用户展现待浏览用户信息的功能，以及界面跳转。

- 用户可左右滑动进行切换浏览，也可以点击左下角“换一个”进行切换浏览。浏览内容包括：待浏览用户的昵称、性别、年龄、星座、学校名称、签名、被赞个数以及标签，若标签的个数大于5个时，点击“更多”按钮，即可来回切换查看标签。
- 点击该界面右上角的三个点，将弹出一个popupwindow，点击popupwindow的不同条目可对应的跳转到相应的界面。

**错误及解决：**

第一次见到这个界面的效果图的时候，大致思路是使用一个Activity，这个Activity的根布局使用FrameLayout(帧布局），帧布局顶层布局标题头、换一个、送玫瑰、赞一个以及用户头像，底层布局一个ViewPager就可实现。但在实现的时候犯了错误：

 - *错误1*：ViewPager的使用文档中建议采用ViewPager+Fragment的方式进行使用，并且FragmentPagerAdapter的重写比较方便，就选择了这种方式。想当然的把用户标签、个人信息放置在了Fragment中，在操作过程中发现效果很是不好：不是只有背景图进行切换，个人信息连同标签都进行了切换；
 - **解决方案**：将用户标签连同个人信息都放置在FrameLayout的顶层（与换一个、送玫瑰、赞一个同级别），选用ViewPager+View的方式，重写MyAdapter继承自PagerADapter、重写四个方法。ViewPager中的View只放置一个ImageView做为背景即可，这样的话在操作的时候就实现了只有背景的切换的效果。

 - *错误2*：在重写PagerAdapter的时候，认为传给Adapter的数据集合中只能是View类型的元素，这样就导致了ViewPager中要展示的View必须在主类中创建并且做好Blur（模糊）工作，这就增加了主类中的代码量，而适配器却仅仅实现了展示的功能；
 - **解决方案**：适配器处理的数据集合不单单可以是View集合，也可以是“真正的”数据集合（例如ListView的适配器接受的是真实数据，在getView方法中进行展示的View的创建以及数据的绑定）。主类在创建ViewPager的适配器的时候，可以只传输一个Bitmap类型的集合，在重写Adapter的instantiateItem方法的时候填充要展示的View，并且取出集合中相对应的Bitmap元素进行模糊，之后将模糊后的图片绑定到Veiw中进行展示。

 - *错误3*：进行虚化操作的时候，只是重写了一个FastBlurUtils类，并调用了doBlur（）方法，发现模糊的效率不是很高，进行滑动的时候有严重的卡顿现象。
 - **解决方案**：在虚化之前，先对要虚化的Bitmap对象进行缩放，长宽压缩的比例越大，压缩之后虚化的效率就越高。

 - *错误4*：一开始构思等待动画的时候，是打算写一个RomanticWaitActivity，然后在该界面进行动画展示并且访问网络的。但毕竟RomanticHomeActivity这个界面才是数据展示的主界面，如果再有RomanticWaitActivity界面的话就涉及到了利用Intent的传值问题，并且传的值不是简单的常见类型的数据，而是自定义复杂类的对象，其中包括的成员变量类型也很复杂。
 - **解决方法**：将等待动画布局布置在RomanticHomeActivity的上层，通过设置Visibility属性来控制动画的展示与隐藏比较方便。但在尝试RomanticWaitActivity向RomanticHomeActivity传值的过程中也学习到了**序列化**对象的很多知识，在它山之石模块进行记录。


**它山之石：**

如何快速虚化一张图片：[高效虚化](http://www.jianshu.com/p/7ae7dfe47a70)

UniversalImageLoader的使用：[图片加载](http://blog.csdn.net/xiaanming/article/details/26810303/)

ViewPager灵敏度：[避免短距离换页](http://blog.csdn.net/wbw1985/article/details/38304171)

ViewPager换页时间：[同步动画时间](http://blog.csdn.net/ekeuy/article/details/12841409)

禁止ViewPager滑动：[强制用户完善信息，否则不能换页](http://www.androidchina.net/3926.html)

序列化知识点：

1. 为什么要进行序列化？答：永久性保存对象，保存对象的字节序列到本地文件中；通过序列化对象在网络中传递对象；通过序列化在进程间传递对象；通过序列化在组件（Activity，Service等）之间传递对象。


2. 序列化的方式有哪些？答：一是实现Serializable接口（是JavaSE本身就支持的），一是实现Parcelable接口（是Android特有功能，效率比实现Serializable接口高效，可用于Intent数据传递，也可以用于进程间通信（IPC））。实现Serializable接口非常简单，声明一下就可以了，而实现Parcelable接口稍微复杂一些，但效率更高，推荐用这种方法提高性能。

3. Parcelable接口是怎样定义的？答：代码如下：

    	public interface Parcelable 
	    {
	    	//内容描述接口，基本不用管
	    	public int describeContents();
	    	//写入接口函数，打包
	    	public void writeToParcel(Parcel dest, int flags);
	    	//读取接口，目的是要从Parcel中构造一个实现了Parcelable的类的实例处理。因为实现类在这里还是不可知的，所以需要用到模板的方式，继承类名通过模板参数传入
	    	//为了能够实现模板参数的传入，这里定义Creator嵌入接口,内含两个接口函数分别返回单个和多个继承类实例
	    	public interface Creator<T> 
		    	{
			    	public T createFromParcel(Parcel source);
			    	public T[] newArray(int size);
		    	}
    	}

4. 实现Serializable接口进行序列化的功能局限是什么？答：加入在RomanticWaitActivity界面中请求了待浏览用户接口，服务器返回给我们若干个用户的信息，在向RomanticHomeActivity传递数据的时候肯定是要传递一个集合，而Bundle传递Serializable对象的方法只有一个putSerializable方法，显然不能传递一个序列化集合；但是在传递Parcelable对象的时候，就有putParcelable（）、putParcelableArrayList（）以及putParcelableArray（）三个方法，不但能够传递序列化对象，还可以传递序列化对象数组及集合。

5. 实现Parcelable接口的示例代码是什么？答：代码如下：

	    public class Book implements Parcelable
    	{
    	private String bookName;
    	private String author;
    	private int publishDate;
    	
    	public Book()
    	{
    	
    	}
    	
    	public String getBookName()
    	{
    	return bookName;
    	}
    	
    	public void setBookName(String bookName)
    	{
    	this.bookName = bookName;
    	}
    	
    	public String getAuthor()
    	{
    	return author;
    	}
    	
    	public void setAuthor(String author)
    	{
    	this.author = author;
    	}
    	
    	public int getPublishDate()
    	{
    	return publishDate;
    	}
    	
    	public void setPublishDate(int publishDate)
    	{
    	this.publishDate = publishDate;
    	}
    	
    	@Override
    	public int describeContents()
    	{
    	return 0;
    	}
    	
    	@Override
    	public void writeToParcel(Parcel out, int flags)
    	{
    	out.writeString(bookName);
    	out.writeString(author);
    	out.writeInt(publishDate);
    	}
    	
    	public static final Parcelable.Creator<Book> CREATOR = new Creator<Book>()
    	{
    	@Override
    	public Book[] newArray(int size)
    	{
    	return new Book[size];
    	}
    	
    	@Override
    	public Book createFromParcel(Parcel in){
    		return new Book(in);
    	}
    	};
    	
    	public Book(Parcel in){
    		bookName = in.readString();
    		author = in.readString();
    		publishDate = in.readInt();
    	}
    	}

6. 当自定义实体类中的属性不是基本数据类型的时候应该怎样处理，比如有集合类型的成员变量，以及数组类型的成员变量？答：[集合类型的成员变量序列化方案](http://2528.iteye.com/blog/1849692)；[数组类型的成员变量序列化方案](http://blog.csdn.net/wenxiang423/article/details/16332217)

7. 非基本类型成员变量序列化示例代码：

		public class RomanticRandomUserBean implements Parcelable {

		private int serializeId;//序列化id
		private String qcCode;//青葱号
		private long birthday;
		private String sex;
		private String schoolName;
		private int greatCounts;
		private List<RomanticUserImageBean> images;//用户照片
		private String nickName;
		private String signature;//签名
		private String constellation;//星座
		private String lables;//标签，多个标签以逗号分隔
		private String[] tagArr;//标签，以字符串数组形式
		    
	    public RomanticRandomUserBean() {
	    }
	    
	    public String[] getTagArr() {
	    return tagArr;
	    }
	    
	    public void setTagArr(String[] tagArr) {
	    this.tagArr = tagArr;
	    }
	    
	    private String status;//用户状态
	    private long createTime;//创建时间
	    
	    public int getSerializeId() {
	    return serializeId;
	    }
	    
	    public void setSerializeId(int serializeId) {
	    this.serializeId = serializeId;
	    }
	    
	    public String getQcCode() {
	    return qcCode;
	    }
	    
	    public void setQcCode(String qcCode) {
	    this.qcCode = qcCode;
	    }
	    
	    public long getBirthday() {
	    return birthday;
	    }
	    
	    public void setBirthday(long birthday) {
	    this.birthday = birthday;
	    }
	    
	    public String getSex() {
	    return sex;
	    }
	    
	    public void setSex(String sex) {
	    this.sex = sex;
	    }
	    
	    public String getSchoolName() {
	    return schoolName;
	    }
	    
	    public void setSchoolName(String schoolName) {
	    this.schoolName = schoolName;
	    }
	    
	    public int getGreatCounts() {
	    return greatCounts;
	    }
	    
	    public void setGreatCounts(int greatCounts) {
	    this.greatCounts = greatCounts;
	    }
	    
	    public List<RomanticUserImageBean> getImages() {
	    return images;
	    }
	    
	    public void setImages(List<RomanticUserImageBean> images) {
	    this.images = images;
	    }
	    
	    public String getNickName() {
	    return nickName;
	    }
	    
	    public void setNickName(String nickName) {
	    this.nickName = nickName;
	    }
	    
	    public String getSignature() {
	    return signature;
	    }
	    
	    public void setSignature(String signature) {
	    this.signature = signature;
	    }
	    
	    public String getConstellation() {
	    return constellation;
	    }
	    
	    public void setConstellation(String constellation) {
	    this.constellation = constellation;
	    }
	    
	    public String getLables() {
	    return lables;
	    }
	    
	    public void setLables(String lables) {
	    this.lables = lables;
	    }
	    
	    public String getStatus() {
	    return status;
	    }
	    
	    public void setStatus(String status) {
	    this.status = status;
	    }
	    
	    public long getCreateTime() {
	    return createTime;
	    }
	    
	    public void setCreateTime(long createTime) {
	    this.createTime = createTime;
	    }
	    
	    
	    @Override
	    public int describeContents() {
	    return 0;
	    }
	    
	    @Override
	    public void writeToParcel(Parcel parcel, int i) {
	    parcel.writeInt(serializeId);
	    parcel.writeString(qcCode);
	    parcel.writeLong(birthday);
	    parcel.writeString(sex);
	    parcel.writeString(schoolName);
	    parcel.writeInt(greatCounts);
	    parcel.writeList(images);
	    parcel.writeString(nickName);
	    parcel.writeString(signature);
	    parcel.writeString(constellation);
	    parcel.writeString(lables);
	    if (tagArr == null) {//如果标签数组为空
	    parcel.writeInt(0);//记录标签数组的长度为0
	    } else {
	    parcel.writeInt(tagArr.length);//记录标签数组的真实长度
	    }
	    if (tagArr != null) {//如果不为空
	    parcel.writeStringArray(tagArr);//将标签数组写入包中
	    }
	    parcel.writeString(status);
	    parcel.writeLong(createTime);
	    }
	    
	    public static final Parcelable.Creator<RomanticRandomUserBean> CREATOR = new Parcelable.Creator<RomanticRandomUserBean>() {
	    @Override
	    public RomanticRandomUserBean createFromParcel(Parcel parcel) {
	    return new RomanticRandomUserBean(parcel);
	    }
	    
	    @Override
	    public RomanticRandomUserBean[] newArray(int i) {
	    return new RomanticRandomUserBean[i];
	    }
	    };
	    
	    private RomanticRandomUserBean(Parcel in) {
	    serializeId = in.readInt();
	    qcCode = in.readString();
	    birthday = in.readLong();
	    sex = in.readString();
	    schoolName = in.readString();
	    greatCounts = in.readInt();
	    images = new ArrayList<RomanticUserImageBean>();
	    in.readList(images, getClass().getClassLoader());
	    nickName = in.readString();
	    signature = in.readString();
	    constellation = in.readString();
	    lables = in.readString();
	    int length = in.readInt();
	    if (length > 0) {
	    tagArr = new String[length];
	    in.readStringArray(tagArr);
	    }
	    status = in.readString();
	    createTime = in.readLong();
	    }
    }

    
**关键代码**：

*获取用户手机屏幕的宽和高：*

    public static int getDisplayScreenWidth(Context context) {
    	WindowManager wmManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
    	return wmManager.getDefaultDisplay().getWidth();
    }

*摒弃之前ViewPager的适配器只是接受View集合的想法，这里适配器接收数据集合，在重写方法的过程中生成或者销毁要展示的view，代码如下：*

	public class RomanticViewPagerAdapter extends PagerAdapter {
	    List<RomanticRandomUserBean> list;
	    Context context;
	
	    public RomanticViewPagerAdapter(List<RomanticRandomUserBean> list, Context context) {
	        this.context = context;
	        this.list = list;
	    }
	
	    @Override
	    public int getCount() {
	        return list.size();
	    }
	
	    @Override
	    public boolean isViewFromObject(View view, Object object) {
	        return view == object;
	    }
	
	    @Override
	    public Object instantiateItem(ViewGroup container, int position) {
	        View view = LayoutInflater.from(context).inflate(R.layout.romantic_home_background, null);
	        ImageView imageView = (ImageView) view.findViewById(R.id.iv_blur_background);
	        Bitmap showBitmap = list.get(position).getImages().get(0).getShowBitmap();
	        if (showBitmap == null) {
	            //如果图片没有加载成功，说明是有其他图片的url与该张图片的url相同导致，这是可以遍历该用户所有图片的url，
	            //找到相同url的图片，判其对应的bitmap是否为空，不为空则取其bitmap
	            //除了第一张图片，遍历每一张图片
	            for (int i = 1; i < list.get(position).getImages().size(); i++) {
	                //如果找到与第一张图片相同的url的图片
	                if (list.get(position).getImages().get(0).getShowImageUrl()
	                        .equals(list.get(position).getImages().get(i).getShowImageUrl())) {
	                    Bitmap bitmap = list.get(position).getImages().get(i).getShowBitmap();
	                    if (bitmap != null) {
	                        list.get(position).getImages().get(0).setShowBitmap(bitmap);
	                        showBitmap = bitmap;
	                        break;
	                    }
	                }
	            }
	        }
	        if(showBitmap!=null){
	            Bitmap scaledBitmap = Bitmap.createScaledBitmap(showBitmap, showBitmap.getWidth() / 6,
	                    showBitmap.getHeight() / 6, false);
	            Bitmap blurBitmap = FastBlurUtils.doBlur(scaledBitmap, 5, false);
	            imageView.setImageBitmap(blurBitmap);
	            container.addView(view);
	            scaledBitmap.recycle();
	        }
	        return view;
	    }
	
	    @Override
	    public void destroyItem(ViewGroup container, int position, Object object) {
	        ((ViewPager) container).removeView((RelativeLayout) object);
	    }

	}


## RomanticJoinActivity界面（修改用户信息）：

<center>
<img src="http://a1.qpic.cn/psb?/V11DxkGh190yEc/dUK0T.qwR1MBfSHTBKaGVRps6jgov33s6Z60cV5fFT4!/b/dP8AAAAAAAAA&bo=gAJxBAAAAAAFB9M!&rf=viewer_4" height="50%"/>
图 J-1. 罗曼蒂克修改用户信息
</center>

**功能简介：**

当用户没有完善信息的时候必须要在该界面完善自己的个人信息，或者当用户需要修改自己在罗曼蒂克模块中的个人信息时（比如修改标签，修改图片等）跳转至该界面。该界面的难点有：流式布局展示标签、照片上传、退出编辑时的圆角提示框、以及星座选择的时候数据的呈现方式。

**错误及解决：**

 - *错误1*：在流式布局中展现“加号”图片的时候，需要为该ImageView添加一个ID，但这个ImageView不在布局文件中，因此无法使用id属性进行定义。当调用该Imageview方法的setId（）方法的时候，形参类型要求为int，定义一个int类型的常量并传入方法中会报如下错误：`Error: Expected resource of type id [ResourceType]`
 - **解决方案**：在报错位置所在的类上面添加此句注解：`@SuppressWarnings("ResourceType")`
 
 - *错误2*：当用户在该界面没有进行保存就打算退出的时候（无论是点击“返回键”或者是左上角“返回箭头”）都应该弹出提示框，如图J-1所示。但实际操作过程中，无论是在布局文件中定义对话框背景为圆角图片、还是圆角自定义图形，出现的效果都是如图J-2所示,四个直角都不能真正的消失。

<center>
<img src="http://a2.qpic.cn/psb?/V11DxkGh190yEc/Jh7hlj3YZ7eXGr1FHjFcEIOjXM5Hi1NGUI9b9vDJHkk!/b/dAkBAAAAAAAA&bo=gAJxBAAAAAAFB9M!&rf=viewer_4" height="50%"/>
图 J-2. 预期效果提示窗口
</center>

<center>
<img src="http://static.oschina.net/uploads/space/2013/0315/095054_L98h_617297.png" height="50%"/>
图 J-3. 错误提示窗口
</center>

 - **解决方案**：在style中定义如下风格:
 
    `<style name="dialog" parent="@android:style/Theme.Dialog">`
   	`<item name="android:windowFrame">@null</item>`
    `<item name="android:windowIsFloating">true</item>`
    `<item name="android:windowIsTranslucent">true</item>`
    `<item name="android:windowNoTitle">true</item>`
    `<item name="android:background">@android:color/transparent</item>`
    `<item name="android:windowBackground">@android:color/transparent</item>`
    `<item name="android:backgroundDimEnabled">true</item>`
    `<item name="android:backgroundDimAmount">0.6</item>`
    `</style>`

其中第六行有background属性，将它的值设置为透明即可。接下来的关键是如何使用该风格：有许多方法使用该风格，包括自定义Dialog等。但最简单的是Java代码中创建对话框的时候调用`new AlertDialog.Builder(this,R.style.dialog).create;`即可使用dialog风格。

- 错误3：当错误2之后，要调用myDialog.setContentView(设置对话框的内容)方法以及myDialog.show(显示对话框)。通常来说（之前使用的时候），往往是先调用setContentView方法再调用show方法。但这里如果按照这个顺序的时候，就会出现如下错误：`requestFeature() must be called before adding content`
- 解决方案：通过上网查阅，出现这个RE（运行时异常）的主要原因由两个，分别是[原因1](http://blog.csdn.net/lvxiangan/article/details/8440206)与setContentView与show方法调用先后顺序的问题，这里先进行show，再调用setContentView方法即可避免该错误。

**它山之石：**

dialog自定义style使用：[方法1](http://my.oschina.net/u/617297/blog/113869)以及[方法2](http://blog.csdn.net/whitley_gong/article/details/50365164)

dialog requestFeature() 异常问题剖析：[方法调用顺序](http://m.blog.csdn.net/article/details?id=43888937)

Android数据选择器：[开源项目PickView](https://github.com/yangchangfu/PickView)

流式布局：[标签自动换行](http://blog.csdn.net/lmj623565791/article/details/38352503/)

**工具代码**：

*重写返回键弹出对话框代码：*

    /**
     * 设置退出对话框的内容
     */
    private void setExitAlertDialog() {
    	//退出编辑提示框内容
		alertDialogView = LayoutInflater.from(this).inflate(R.layout.romantic_exit_alertdialog, null);
		TextView tvCancle = (TextView) alertDialogView.findViewById(R.id.tv_cancel);
		TextView tvEnsure = (TextView) alertDialogView.findViewById(R.id.tv_ensure);
		tvCancle.setOnClickListener(mOnClickListener);
		tvEnsure.setOnClickListener(mOnClickListener);
		exitAlertDialog = new AlertDialog.Builder(this, R.style.dialog).create();
    }
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
	    if (keyCode == KeyEvent.KEYCODE_BACK && event.getRepeatCount() == 0) {
	    	if (exitAlertDialog != null) {
	    		exitAlertDialog.show();
	    		exitAlertDialog.setContentView(alertDialogView);
	    	}
	    	return true;
	    } else {
	    	return super.onKeyDown(keyCode, event);
	    }
    }

*数据选择器PickView的使用：*

    /**
     * 星座_点击监听
     */
    @OnClick(R.id.ll_constellation)
    void constellationListener(View view) {
	    String[] constellations = {"白羊座", "金牛座", "双子座", "巨蟹座", "狮子座", "处女座", "天秤座", "天蝎座", "射手座", "摩羯座", "水瓶座", "双鱼座"};
	    List<Item> items = new ArrayList<>();
	    for (int i = 0; i < constellations.length; i++) {
		    Item item = new Item();
		    item.name = constellations[i];
		    items.add(item);
	    }
	    PickView pickView = new PickView(ctx);
	    pickView.setPickerView(items, PickView.Style.SINGLE);
	    pickView.setOnSelectListener(new PickView.OnSelectListener() {
	    	@Override
	    	public void OnSelectItemClick(View view, int[] selectedIndexs,
	      	String selectedText) {
	    		tvConstellation.setText(selectedText);
	    	}
	    });
	    pickView.show();
    }

*点击保存的时候需要访问两个接口，只有当两个接口都访问成功的时候，才自动返回主界面，判定两个接口都成功的代码为：*

    void createTimerThread() {
    	if (mTimer == null) {
    		mTimer = new Timer();
    	}
    	if (mTask == null) {
    		mTask = new TimerTask() {
   				@Override
    			public void run() {
    				if (isSignSave && isNickNameSave) {
    					handler.sendEmptyMessage(SAVE_SUCCESS);
    					cancleTimerThread();
   					}	
    			}
    		};
    	}
    	mTimer.schedule(mTask,0,100);
    }
    
    void cancleTimerThread() {
    	if (mTask != null) {
    		mTask.cancel();
    		mTask = null;
    	}
    	if (mTimer != null) {
    		mTimer.cancel();
    		mTimer = null;
    	}
    }


## RomanticSearchActivity界面（修改待浏览用户的条件）：

<center>
<img src="http://a3.qpic.cn/psb?/V11DxkGh190yEc/IBTWePcdHGTSZ.cdHiQd1EqFnh9.eG3cLHBeH75TUxI!/b/dHABAAAAAAAA&bo=0AIABQAAAAAFB*M!&rf=viewer_4" height="50%"/>
图 S-1. 修改范围界面
</center>

**功能简介：**
该界面用来满足用户浏览不同地区、不同学校、不同性别的其他用户的信息。

**错误及解决：**

 - *错误1*：当用户所选的学校确定时，又选择了城市，如果选择的城市和之前的学校根本不匹配时（比如学校一栏显示的是郑州大学，用户又选择了城市列表里的武汉）点击保存，就会出现逻辑错误。
 - **解决方案**：当选择的条目之间具有范围所属关系的时候，当选择大范围的条目的时候，应将小范围的条目清空。这里当选择城市的时候，如果城市与学校具备所属关系，学校可以继续存在；但当城市发生变化的时候，应当将学校一栏清空，并在保存的时候进行非空判断。

**它山之石：**

在非activity中(比如在popupwindow中)调用startActivityForResult，[onActivityResult方法在哪里重写](http://www.360doc.com/content/11/0720/10/7322578_134657348.shtml)



