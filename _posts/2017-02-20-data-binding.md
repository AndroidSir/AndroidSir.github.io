---
layout: post
title:  "Data Binding Library"
date:   2017-02-20
categories: 知识点
tags: Data-Binding
---

* content
{:toc}

本文档说明如何使用数据绑定库来编写声明式布局，并使应用程序逻辑和布局所必需的粘合代码最小化。该库兼容至Android 2.1。




## 编译环境

首先在Android SDK管理器中下载该支持库。然后在app module的build.gradle文件中增加如下代码：

	android {
	    ....
	    dataBinding {
	        enabled = true
	    }
	}

即使app module已经引用了一个使用数据绑定库的第三方库，你仍然需要为主app module设置上述代码。

## 数据绑定布局文件

### 编写第一组数据绑定表达式

数据绑定布局文件和常规的布局文件有以下不同：

1. root节点为layout
2. 次root节点为data
3. data节点需要有variable子节点
4. data节点之后才是常规布局

实例代码如下：

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android">
	   <data>
	       <variable name="user" type="com.example.User"/>
	   </data>
	   <LinearLayout
	       android:orientation="vertical"
	       android:layout_width="match_parent"
	       android:layout_height="match_parent">
	       <TextView android:layout_width="wrap_content"
	           android:layout_height="wrap_content"
	           android:text="@{user.firstName}"/>
	       <TextView android:layout_width="wrap_content"
	           android:layout_height="wrap_content"
	           android:text="@{user.lastName}"/>
	   </LinearLayout>
	</layout>

data节点内的variable节点描述了这个布局会用到的属性名：

	<variable name="user" type="com.example.User"/>

接下来的常规布局的属性值采用`@{}`语法描述。比如：

	<TextView
		 android:layout_width="wrap_content"
	     android:layout_height="wrap_content"
	     android:text="@{user.firstName}"/>

###　数据对象

	public class User {
	   public final String firstName;
	   public final String lastName;
	   public User(String firstName, String lastName) {
	       this.firstName = firstName;
	       this.lastName = lastName;
	   }
	}

上述类型的对象的属性值不会改变，也可以使用如下类，是其属性值可以发生改变：

	public class User {
	   private final String firstName;
	   private final String lastName;
	   public User(String firstName, String lastName) {
	       this.firstName = firstName;
	       this.lastName = lastName;
	   }
	   public String getFirstName() {
	       return this.firstName;
	   }
	   public String getLastName() {
	       return this.lastName;
	   }
	}

从数据绑定的角度来看，上述两个类是平等的的。 `@{user.firstName}`会使用第一个类的`firstName`成员变量，会使用第二个类的`getFirstName()`方法来获取属性值。另外，如果`firstName()`方法存在的话，该方法也会被调用。

###　数据绑定

默认的，Binding类的生成将依赖于布局文件的名字，将布局文件的名字中的每一个单此采用首字母大写并连接，且添加Binding后缀。如`main_activity.xml`文件对应着`MainActivityBinding`类。该类持有了从属性到布局视图的所有绑定，且知道如何通过绑定表达式将属性值赋予给对应的View。简单代码示例如下：

	@Override
	protected void onCreate(Bundle savedInstanceState) {
	   super.onCreate(savedInstanceState);
	   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
	   User user = new User("Test", "User");
	   binding.setUser(user);
	}

如果在ListView或Recyclerview适配器内部为Item绑定数据，需要使用：

	ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup, false);
	//or
	ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);

### 事件处理

Data Binding允许我们处理有View分发的事件。事件属性名称受监听方法的名称的约束，有少数例外。例如，`View.OnLongClickListener`有一个`onLongClick()`方法,因此针对该事件的属性名称为：`android:onLongClick`。有两种方法来处理事件：

1. Method References（方法引用）：在表达式中，可以引用符合侦听器方法签名的方法。当表达式的值为一个方法的引用的时候，数据绑定在监听器中封装方法引用和所有者对象，并为目标view设置该监听。如果表达式值为null，数据绑定不会创建监听，也不会为相应的view设置监听。
2. Listener Binding（监听绑定）：当事件发生的时候存在lambda表达式。数据绑定总是创建一个监听器，并设置给view.当事件被分发的时候，监听器将计算lambda表达式。

#### Method References（方法引用）

事件可以直接绑定到处理方法，就好像`android:onClick`属性可以匹配Activity的相应方法一样。较之View的onClick而言的一大优势是表达式在编译时处理，因此如果方法不存在或者方法的签名不正确，你将会受到一个编译时错误。

方法引用和监听绑定的最大不同在于前者真实的监听实现是当数据绑定的时候就创建的，而不是当事件被触发。如果你更喜欢当事件发生的时候计算表达式的值，你应该使用监听绑定。

若要将事件分配给其处理程序，请使用常规绑定表达式，表达式的值是要调用的方法名。例如，如果您的数据对象有如下方法：

	public class MyHandlers {
	    public void onClickFriend(View view) { ... }
	}

绑定表达式可以按如下方式为视图分配单击侦听器：

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android">
	   <data>
	       <variable name="handlers" type="com.example.Handlers"/>
	       <variable name="user" type="com.example.User"/>
	   </data>
	   <LinearLayout
	       android:orientation="vertical"
	       android:layout_width="match_parent"
	       android:layout_height="match_parent">
	       <TextView android:layout_width="wrap_content"
	           android:layout_height="wrap_content"
	           android:text="@{user.firstName}"
	           android:onClick="@{handlers::onClickFriend}"/>
	   </LinearLayout>
	</layout>

注意表达式中的方法的签名必须与侦听对象中的方法的签名完全匹配。

#### 监听绑定

侦听器绑定是绑定在事件发生时运行的表达式。这和方法引用是相似的，但是这允许运行任意的数据绑定表达式。该特性需在Gradle 2.0及更高版本使用。

在使用方法引用时，方法的参数必须和事件监听器的参数相匹配。但使用监听绑定，只需要方法的返回值和监听器的返回值相同即可（void除外）。例如，一个Presenter类可以有如下方法：

	public class Presenter {
	    public void onSaveClick(Task task){}
	}

我们可以按如下方式绑定监听：

	<?xml version="1.0" encoding="utf-8"?>
	  <layout xmlns:android="http://schemas.android.com/apk/res/android">
	      <data>
	          <variable name="task" type="com.android.example.Task" />
	          <variable name="presenter" type="com.android.example.Presenter" />
	      </data>
	      <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
	          <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
	          android:onClick="@{() -> presenter.onSaveClick(task)}" />
	      </LinearLayout>
	  </layout>

监听由lambda代表，且该lambda表达式只被允许作为表达式的root元素。当一个回调使用该表达式的时候，Data Binding将自动为该事件创建并注册一个必要的监听器。当视图触发事件时，数据绑定将计算给定的表达式。与常规绑定表达式一样，在计算这些侦听表达式时，将得到null和数据绑定的线程安全性。

注意上边的代码，我们没有定义view的参数并赋值给 `onClick(android.view.View)`方法。Listener binding为监听参数提供了两个选择：既可以忽略该方法的所有参数，也可以仅仅忽略它们的名字。如果你喜欢参数的名字，你可以在表达式中使用她们。例如，上边的表达式也可以写成：

	android:onClick="@{(view) -> presenter.onSaveClick(task)}"

否则如果你想要使用有参表达式，也可以如下：

	public class Presenter {
	    public void onSaveClick(View view, Task task){}
	}

且：

	android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"

你也可以使用多参数的Lambda表达式：

	public class Presenter {
	    public void onCompletedChanged(Task task, boolean completed){}
	}

且：

	  <CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
	        android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />

如果你监听的事件的返回值非void，你的表达式必须返回一个同类型的值。例如，如果你要监听的时间是长点击事件，你的表达式应该返回boolean：

	public class Presenter {
	    public boolean onLongClick(View view, Task task){}
	}

且：

	android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"

如果由于对象为null导致表达式不能被计算，Data Binding将返回数据类型的默认值，如引用类型返回null，int型返回0，boolean类型返回false。

如果你需要一个判断表达式（如三目运算符），你可以使用void作为一个符号：

	android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"

#### 避免复杂的监听

监听表达式非常强大，可以让你的代码更加便于阅读。但相反，若监听中包含复杂的表达式则会让你的布局文件难于阅读，且不利于后期维护。这些表达式应该是简单的，从用户界面传递可用数据到回调方法。你应该在监听表达式调用的回调方法中实现任意的业务逻辑。

为了避免冲突，一些特殊的点击事件不使用`android:onClick`属性如下表所示：

...此处省略一个表...

## 布局细节

### Imports

在data元素中可以使用一个或多个import元素。让向布局文件中引用类更加的简单，就像在使用java一样：

	<data>
	    <import type="android.view.View"/>
	</data>

现在，View就可以在binding表达式中使用了：

	<TextView
	   android:text="@{user.lastName}"
	   android:layout_width="wrap_content"
	   android:layout_height="wrap_content"
	   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>

当存在重名的情况，其中一个类需要通过`alias`重命名：

	<import type="android.view.View"/>
	<import type="com.example.real.estate.View"
	        alias="Vista"/>

现在，Vista代表`com.example.real.estate.View`，而View代表`android.view.View`。

导入的类型可以用作变量和表达式中的类型引用：

	<data>
	    <import type="com.example.User"/>
	    <import type="java.util.List"/>
	    <variable name="user" type="User"/>
	    <variable name="userList" type="List&lt;User&gt;"/>
	</data>

> 注意：Android Studio还没有处理导入，因此为变量自动导入还不能够实现。对于这个问题，你可以通过为变量使用全类名，你的应用将正确的编译。

	<TextView
	   android:text="@{((User)(user.connection)).lastName}"
	   android:layout_width="wrap_content"
	   android:layout_height="wrap_content"/>

在表达式中使用导入类型的静态字段和静态方法：

	<data>
	    <import type="com.example.MyStringUtils"/>
	    <variable name="user" type="com.example.User"/>
	</data>
	…
	<TextView
	   android:text="@{MyStringUtils.capitalize(user.lastName)}"
	   android:layout_width="wrap_content"
	   android:layout_height="wrap_content"/>

### 变量（Variables）

在data元素中可以使用任意数量的变量。每一个变量元素描述一个属性，该属性将在binding表达式中使用。

	<data>
	    <import type="android.graphics.drawable.Drawable"/>
	    <variable name="user"  type="com.example.User"/>
	    <variable name="image" type="Drawable"/>
	    <variable name="note"  type="String"/>
	</data>

变量类型在编译的时候将被检查，因此如果一个变量实现了 `Observable`或一个 `observable collection`，在类型中应该反映出来。如果一个变量是一个基类或接口，没有实现`Observable`接口，这个变量将不会被观察。

当为不同设备（横向和竖向）准备不同的布局文件时，变量将被合并。这些布局文件之间不能有冲突的变量定义。

生成的binding类将为存在对应于每一个引入的变量的setter和getter方法。在setter方法被调用之前，这些变量将会获得java默认值。

当绑定表达式需要使用context的时候，名为context的变量被生成。该变量的值来自于root的`getContext()`方法。

### 自定义绑定类名

默认的，Binding类的名称取决于布局文件的文件名。首字母大写、去掉下划线并添加“Binding”后缀。该类被置于module包下的databinding包下。

通过修改data元素的class属性，Binding类可以被重命名或置于其它包下。如：

	<data class="ContactItem">
	    ...
	</data>

将会在当前module包下的databinding包下生成名为ContactItem的Binding类。该类也可以置于不同包下，如：

	<data class=".ContactItem">
	    ...
	</data>

此时，名为ContactItem的Binding类将直接生成与当前module包下。当然，该类可以被置于任意包下，如：

	<data class="com.example.ContactItem">
	    ...
	</data>

### Includes(包含)

通过使用应用程序命名空间和属性中的变量名，可以将变量从原来布局中传递到现在的布局中：

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android"
	        xmlns:bind="http://schemas.android.com/apk/res-auto">
	   <data>
	       <variable name="user" type="com.example.User"/>
	   </data>
	   <LinearLayout
	       android:orientation="vertical"
	       android:layout_width="match_parent"
	       android:layout_height="match_parent">
	       <include layout="@layout/name"
	           bind:user="@{user}"/>
	       <include layout="@layout/contact"
	           bind:user="@{user}"/>
	   </LinearLayout>
	</layout>

此时，必须有name.xml和contact.xml两个布局文件。

Data binding不支持将包含作为merge元素的直接子标签，因此，下面的布局是错误的：

	<?xml version="1.0" encoding="utf-8"?>
	<layout xmlns:android="http://schemas.android.com/apk/res/android"
	        xmlns:bind="http://schemas.android.com/apk/res-auto">
	   <data>
	       <variable name="user" type="com.example.User"/>
	   </data>
	   <merge>
	       <include layout="@layout/name"
	           bind:user="@{user}"/>
	       <include layout="@layout/contact"
	           bind:user="@{user}"/>
	   </merge>
	</layout>

### Expression Language（表达式语言）

#### 共性

表达式语言和Java表达式有很多相似之处。例如：

1. 算术运算符：+-*/%
2. 字符串拼接：+
3. 逻辑运算符： && ||
4. 二进制运算符： & | ^
5. 弹幕运算符：+-!~
6. 唯一运算符：>>  >>>  <<
7. 比较运算符：== > < >= <=
8. instanceof
9. 分组：（）
10. 字面值：character，String，numeric，null
11. Cast
12. Method calls
13. Field access
14. Array access []
15. 三目运算符：？：

例如：

	android:text="@{String.valueOf(index + 1)}"
	android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
	android:transitionName='@{"image_" + id}'

#### 不支持的操作

一些操作在表达式语言中并不支持：

1. this
2. super
3. new
4. 明确的通用的调用

#### 判null操作

判null操作运算符"??",当非null时选择执行运算符左端，否则（null）时执行运算符右侧。如：

	android:text="@{user.displayName ?? user.lastName}"

等同于：

	android:text="@{user.displayName != null ? user.displayName : user.lastName}"

#### 属性引用

当在表达式中引用一个类的属性的时候，使用成员变量，getter和可观察的变量的作用是一致的。如：

	android:text="@{user.lastName}"

#### 避免空指针异常

生成数据绑定代码会自动检查null和避免空指针异常。例如，在表达式`@{user.name}`中，如果user为null，则user.name将会是默认值null。如果同时使用了int类型的user.age，它将会是默认值0.

#### Collections（集合）

可用集合：arrays, lists, sparse lists, maps, 方便起见可以使用[]，如：

	<data>
	    <import type="android.util.SparseArray"/>
	    <import type="java.util.Map"/>
	    <import type="java.util.List"/>
	    <variable name="list" type="List&lt;String&gt;"/>
	    <variable name="sparse" type="SparseArray&lt;String&gt;"/>
	    <variable name="map" type="Map&lt;String, String&gt;"/>
	    <variable name="index" type="int"/>
	    <variable name="key" type="String"/>
	</data>
	…
	android:text="@{list[index]}"
	…
	android:text="@{sparse[index]}"
	…
	android:text="@{map[key]}"

#### 字符串字面值

表达式中使用双引号代表字符串的时候，属性值应使用单引号包裹：

	android:text='@{map["firstName"]}'

当然，属性值也可以被双引号包裹，当这样组偶的时候，字符串字面值应该使用单引号或反引号：

	android:text="@{map[`firstName`}"
	android:text="@{map['firstName']}"

### Resources

允许使用正常语法将资源作为表达式的一部分：

	android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"

格式字符串和复数可能可以通过提供的参数：

	android:text="@{@string/nameFormat(firstName, lastName)}"
	android:text="@{@plurals/banana(bananaCount)}"

## 数据对象

任何普通的java对象（POJO）可用于数据绑定，但修改一个POJO不会造成UI更新。数据绑定可以具备在数据变化时进行通知的能力。有三种不同的数据更改通知机制，可观察对象，可观察字段和可观察集合。当这些可观察数据对象绑定到UI时，当数据对象的属性发生改变，会导致用户界面将自动更新。

### Observable Objects（可观察对象）

实现了`Observable`接口的类将允许绑定将单个监听器对应到绑定对象以监听该对象上的所有属性的更改。

可观察到的接口有一个机制来添加和删除监听器，但具体通知是由开发人员完成的。为使开发更容易，系统提供了一个基类Baseobservable，该类是为了实现侦听器注册机制。数据类实现者仍然可以负责通知属性的变化。这是通过为getter和setter分配一个可绑定的注解进行通知。如：

	private static class User extends BaseObservable {
	   private String firstName;
	   private String lastName;
	   @Bindable
	   public String getFirstName() {
	       return this.firstName;
	   }
	   @Bindable
	   public String getLastName() {
	       return this.lastName;
	   }
	   public void setFirstName(String firstName) {
	       this.firstName = firstName;
	       notifyPropertyChanged(BR.firstName);
	   }
	   public void setLastName(String lastName) {
	       this.lastName = lastName;
	       notifyPropertyChanged(BR.lastName);
	   }
	}

### ObservableFields（可观察的成员变量）

在创建可观察类的时候将做一些工作，因此想要节省时间或者仅有很少的属性的时候可以使用 `ObservableField`类和与它相似的类：`ObservableBoolean, ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble,  ObservableParcelable` 

`ObservableField`是有单一成员变量的独立的可观察对象。原始版本避免在装箱和拆箱过程中进行访问操作。若要使用，请在数据类中创建公共最终字段，如：

	private static class User {
	   public final ObservableField<String> firstName =
	       new ObservableField<>();
	   public final ObservableField<String> lastName =
	       new ObservableField<>();
	   public final ObservableInt age = new ObservableInt();
	}

就是这样，为了获取属性的值，可以通过set和get方法：

	user.firstName.set("Google");
	int age = user.age.get();

### Observable Collections（可观察集合）

一些应用程序更多的使用动态结构来保存数据。可观察的集合允许通过键访问的方式访问这些数据对象。当key是引用类型，如String时，`ObservableArrayMap`类的作用非常大：

	ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
	user.put("firstName", "Google");
	user.put("lastName", "Inc.");
	user.put("age", 17);

在布局中，该map可以通过String类型的key被使用：

	<data>
	    <import type="android.databinding.ObservableMap"/>
	    <variable name="user" type="ObservableMap&lt;String, Object&gt;"/>
	</data>
	…
	<TextView
	   android:text='@{user["lastName"]}'
	   android:layout_width="wrap_content"
	   android:layout_height="wrap_content"/>
	<TextView
	   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
	   android:layout_width="wrap_content"
	   android:layout_height="wrap_content"/>

当key值是int类型的时候，`ObservableArrayList`是有用的：

	ObservableArrayList<Object> user = new ObservableArrayList<>();
	user.add("Google");
	user.add("Inc.");
	user.add(17);

在布局中，上述集合可以通过索引进行使用：

	<data>
	    <import type="android.databinding.ObservableList"/>
	    <import type="com.example.my.app.Fields"/>
	    <variable name="user" type="ObservableList&lt;Object&gt;"/>
	</data>
	…
	<TextView
	   android:text='@{user[Fields.LAST_NAME]}'
	   android:layout_width="wrap_content"
	   android:layout_height="wrap_content"/>
	<TextView
	   android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
	   android:layout_width="wrap_content"
	   android:layout_height="wrap_content"/>

## Generated Binding（创建Binding）

生成的绑定类将布局变量与布局内的视图连接起来。如前所述，生成的Binding类可以自定义类名及其所属的包。生成的Binding类都是`ViewDataBinding`的子类。

### Creating(创建)

绑定应在填充后不久创建，以确保视图层次结构在将表达式绑定到视图之前不会受到干扰。有不同的方法来bind一个布局。最常用的是结合填充视图层次的方法去使用Binding类的静态方法。这有一个更简单的版本，只需要一个LayoutInflater和一个需要绑定的ViewGroup：

	MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater);
	MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater, viewGroup, false);

如果该布局被其他填充机制填充，则可以单独绑定：

	MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);

有时绑定不能事先知道。这种情况下，绑定可以通过`DataBindingUtil`类完成：

	ViewDataBinding binding = DataBindingUtil.inflate(LayoutInflater, layoutId,
	    parent, attachToParent);
	ViewDataBinding binding = DataBindingUtil.bindTo(viewRoot, layoutId);

### Views With IDs（View的id）