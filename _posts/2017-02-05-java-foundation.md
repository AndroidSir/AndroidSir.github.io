---
layout: post
title:  "Java 基础"
date:   2017-02-05
categories: 知识点
tags: final 堆栈 多态 泛型
---

* content
{:toc}

本文主要记录一些零碎的Java基础知识。




## 代码编写总则

分析： 从具体到抽象

实现：从抽象到具体

写一个方法应该明确两件事情：1.明确返回值类型  2. 明确参数列表

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

## 多态

形成多态的条件：

1. 继承
2. 方法重写
3. 父类引用指向子类对象

多态中的成员访问特点：

1. 成员变量：编译看左边，运行看左边（没有成员变量的重写）
2. 构造方法：创建子类对象的时候，访问父类的构造方法，对父类的数据进行初始化
3. 成员方法：编译看左边，运行看右边
4. 静态方法：编译看左边，运行看左边（静态和类相关，算不上重写，所以，访问还是左边的）

多态的弊端：不能使用子类的特有功能。   

多态的分类：

1. 具体类多态（几乎不用）
2. 抽象类多态（常用）
3. 接口多态（最常用）                                                                                                                                                                                                                                                                                                                     

多态的例子：
	
	class A {
	        public void show() {
	            show2();
	        }
	
	        public void show2() {
	            System.out.println("我");
	        }
	    }
	
	    class B extends A {
	        public void show2() {
	            System.out.println("爱");
	        }
	    }
	
	    class C extends B {
	        public void show() {
	            super.show();
	        }
	
	        public void show2() {
	            System.out.println("你");
	        }
	    }
	
	    public class DuoTaiTest {
	        public static void main(String[] args) {
	            A a = new B();
	            a.show();
	            B b = new C();
	            b.show();
	        }
	    }

通过运行该例子，得到的结果是“爱你”。分析如下：

a.show():运行的其实是B类对象的show（）方法，而类B没有重写类A的show（）方法，因此B类对象的show（）方法是继承自A类的，用于执行show2（）方法，因此最终结果是通过B类的show（）方法，调用B类的show2（）方法。

b.show():运行的是C类对象的show（）方法，而C类对象的show（）方法调用的是B类对象的show（）方法，而B类对象的show()方法是去调用show2（）方法，那么这个show2（）方法是C的还是B的呢？结果显示是C类的show2（）方法被调用，为什么呢，看下边的代码：

		class A {
	        public void ele() {
	        	//System.out.println(this.getClass().getSimpleName()+"ele");
				//quchifan();//等同于下一句
	            this.quchifan();
	        }
	
	        public void quchifan() {
	            System.out.println("eat water");
	        }
	    }
	
	    class B extends A {
			/*public void ele() {
	        	//System.out.println(this.getClass().getSimpleName()+"ele");
				//quchifan();//等同于下一句
	            this.quchifan();
	        }*/

	        public void quchifan() {
	            System.out.println("eat xiaomi");
	        }
	    }
	
	    class C extends B {
	        public void ele() {
	            super.ele();//儿子饿了，问爸爸饿了要怎么做，爸爸说：饿了要吃饭。因此儿子就去吃饭了。不是说儿子饿了，就对爸爸说：爸爸，我都饿了，你也必须饿。
				//this.quchifan();
	        }
	
	        public void quchifan() {
	            System.out.println("eat dami");
	        }
	    }
	
	    public class DuoTaiTest2 {
	        public static void main(String[] args) {
	            A a = new B();
	            a.ele();
	            B b = new C();
	            b.ele();
	        }
	    }

B类继承A类的时候，没有重写ele()方法，因此B类就从A类继承了public的ele()方法，如B类注释所示，从结果可以发现**父类的this粘贴到子类的时候this是不会变为super的，而依然是this**。因此C类的ele()方法体的两句话其实是等价的，第二句话就是将父类的ele（）方法体粘贴过来的结果，也就是说，父类的方法调用可以认为是父类方法体的粘贴。

**结论**：让子类具有和父类一样的public方法的时候（方法继承）可以有如下三种方法（一般来说三者等价）：

1. 不写，系统默认继承；
2. 将父类的方法签名及方法体完全复制到子类中，父类中省略的this到子类中依然是this而非super；（最容易理解）
3. 重写方法时**仅仅**通过super关键字调用父类的同名方法，此时不应该叫方法重写，而是方法继承。

什么时候不等价呢？当父类的方法体中使用了私有的属性或方法的时候，子类是不能够通过复制父类的方法签名及方法体进行方法继承（非重写）的，此时只有1和3两种办法进行方法继承。代码如下：

1. 若父类的public方法中使用到了父类的private成员变量及父类的public方法，子类继承该public方法的时候，使用的是**父类**private成员变量以及**自己**继承的public方法。除非子类定义同名的成员变量。
2. 若父类的public方法中使用到了父类的private和public方法，子类继承该public方法的时候，使用的是**父类**的private方法以及**自己**继承的public方法。

类似于模范方法模式，父类规定好了若干方法执行的流程，其中一些方法可以让子类自由实现，其中关键的方法则是父类已经定义好的，不能被子类修改的。当子类重写好了可以重写的方法之后，让父类执行规定的流程，父类则会顺序调用子类实现的方法以及自己定义好的不可让子类自由实现的方法。

## 抽象类

抽象**类的特点**：

1. 抽象类和抽象方法必须用abstract关键字修饰；
2. 抽象类中不一定有抽象方法，但是有抽象方法的类必须定义为抽象类；
3. 抽象类不能被实例化：
	+ 因为它不是具体的；
	+ 抽象类不能实例化，但是有构造方法，用途是：用于子类访问父类数据的初始化
4. 抽象类的子类：
	+ 如果不想重写抽象方法，该子类仍是一个抽象类；
	+ 若重写了所有的抽象方法，这个时候子类就是一个具体的类了，可以不用abstract关键字来修饰。

抽象类的**成员特点**：

1. 成员变量：既可以是变量，也可以是常量；
2. 构造方法：用于子类访问父类数据的初始化；
3. 成员方法：既可以是抽象的，也可以是非抽象的。

抽象类的成员**方法特点**：

1. 抽象方法：强制要求子类做的事情
2. 非抽象方法：子类继承的事情，提高代码复用性。

问：一个类如果没有抽象方法，可不可以定义为抽象类？如果可以，有什么意义？

答：可以，不让创建对象（构造方法私有化绝大数情况是为了单例的需要，而非不让实例化）

abstract不能可哪些关键字共存？

	private 冲突
	
	final 冲突
	
	static 无意义

## 接口

接口的作用：**扩展**与**规范**

问：接口如何实例化？   答：按照多态的方式进行接口的实例化。、

接口的子类：

1. 可以使具体类，该类要重写接口中的所有抽象方法；
2. 可以使抽象类，但是意义不大：如下代码

	abstract MyClass implement MyInterface{
		//MyInterface中定义了很多抽象方法，但是MyClass但不用重写，且不会报错
	}

接口成员特点：

1. 成员变量：只能是常量，并且是静态的；默认修饰符是`public static final`；建议自己手动给出修饰符。
2. 成员方法：
	+ Java8之前：只能是抽象方法；默认修饰符是`public abstract`；建议自己手动给出修饰符
	+ Java8：[在接口中通过static和default修饰符修饰的方法可以实现方法体](http://www.cnblogs.com/kaiblog/p/5295873.html)
3. 构造方法：接口没有构造方法。
	+ 既然接口没有构造方法，那么接口的实现类在构造方法中调用`super()`的作用是什么呢？答：调用的是Object类的构造方法。因为所有的类都默认继承自Object，且Object只有一个`无参的构造方法`。

## 接口和抽象类的区别

成员变量：

1. 抽象类：
	+ 成员变量：可以变量，也可以常量
	+ 构造方法：有
	+ 成员方法：可以抽象，也可以非抽象
2. 接口：
	+ 成员变量：只可以是常量
	+ 构造方法：无
	+ 成员方法：在Java8之前只能是抽象方法，在Java8之后通过default和static关键字修饰的方法可以包含方法体（具体方法）。

关系区别：

1. 类与类：单继承
2. 类与接口：单实现，多实现
3. 接口与接口：单继承，多继承

设计理念的区别：

1. 抽象类被继承体现的是“is a”的关系。抽象类中定义的是该继承体系的共性功能。
2. 接口被实现体现的是“like a （can）”的关系。接口中定义的是该继承体系的扩展功能。

## 泛型（参数化类型）

泛型(Generic)，即参数化类型（parameterised type）,允许程序在创建集合时指定集合元素的类型。如果使用带泛型的接口、类定义变量，那么调用构造器创建对象时构造器之后不需要带完整的泛型信息，只要给出一对尖括号（<>）即可。

所谓泛型，就是允许在定义类、接口、方法时使用类型形参，这个类型形参将在声明变量、创建对象、调用方法时动态地指定（即传入实际的类型参数，也可以称为类型实参）。

### 特点

定义构造器的时候，构造器名还是原来的类名，调用该构造器的时候可以使用菱形语法，为形参传入实际的类型参数。

调用方法时必须为所有的数据形参传入参数值，与调用方法不同的是，使用类、接口时也可以不为类型形参传入实际的类型参数。

注意：

+ 如果Foo是Bar的一个子类型（子类或者子接口），而G是具有泛型声明的类或接口，`G<Foo>`并不是`G<Bar>`的子类型。

+ **String是Object的子类型，但`List<String>`并不是`List<Object>`的子类型。**

+ 数组和泛型有所不同，假设Foo是Bar的一个子类型（子类或者子接口），那么Foo[]依然是Bar[]的子类型。

### 类型通配符及类型形参的上限

当无法确定类型实参时，可以使用类型通配符`？`做为类型实参。

使用类型通配符的时候，可以设定类型通配符的上限。如`List<? extends UpperBoundClass>`

定义类型形参的时候，也可以为类型形参设定上线。如`public class Apple<T extends Number>{}`

当类型形参需要设定多个上限的时候（至多有一个父类上限，可以有多个接口上限），表明该类型形参必须使其父类的子类（父类本身也行），并且实现多个上限接口。此时，若存在类上限，类上限必须位于第一位。代码如下：

    public class Apple<T extends Number & java.io.Serializable>{}

### 泛型方法

在定义类、接口时可以使用类型形参，在该类的方法定义和成员变量定义、接口的方法定义中，这些类型形参可以被当成普通类型来用。在另外一些情况下，定义类、接口时没有使用类型形参，但定义方法时想自己定义类型形参，这也是可以的。使用格式如下：

	修饰符 <T , S> 返回值类型  方法名（形参列表）{
		// 方法体
	}

如：

    static <T> void fromArrayToCollection(T[] a,Collection<T> c){
    	for(T o:a){
    		c.add(o);
    	}
    }

### 泛型方法与类型通配符的区别

大多数时候都可以使用泛型方法来代替类型通配符。如：

	public interface Collection<E>{
		boolean containAll(Collection<?> c);
		boolean addAll(Collection<? extends E> c);
		...
	}

也可以这样实现

	public interface Collection<E>{
		<T> boolean containAll(Collection<T> c);
		<T extends E> boolean allAll(Collection<T> c);
		...
	}

上面两个方法中类型形参T只使用了一次，类型形参T产生的唯一效果是可以在不同的调用点传入不同的实际类型。对于这种情况，应该使用通配符：通配符就是被设计用来支持灵活的子类化的。

当方法的一个或多个参数之间的类型存在依赖关系，或者方法返回值与参数之间的类型存在依赖关系，应使用泛型方法。如果没有这样的类型依赖关系，就不应该使用泛型方法。

### 泛型构造器

Java允许在构造器签名中声明类型形参，这样就产生了所谓的泛型构造器。

	class Foo{
		public <T> Foo(T t){
			System.out.println(t);
		}
	}

在调用泛型构造器的时候，不仅可以让Java根据数据参数的类型来“推断”类型形参的类型，也可以显式地为构造器中的类型形参指定实际的类型。如：

	new Foo(200);
	new Foo("疯狂Java讲义");
	new <String> Foo("疯狂Java讲义");//正确
	new <String> Foo(12.3);//错误

显式指定了泛型构造器中声明的类型形参的实际类型时，不可以使用菱形语法(属于类的定义泛型)，如：

	class MyClass<E>{
		public <T> MyClass(T t){
		System.out.println(t);
		}
	}

上述类的定义中存在两个泛型：

	MyClass<String> mc1 = new MyClass<>(5);//正确，没有显式指定泛型构造器的实际类型，即同时不指定
	MyClass<String> mc2 = new <Integer> MyClass<String>(5);//正确，同时指定
	MyClass<String> mc3 = new <Integer> MyClass<>(5);//错误，如果显式指定泛型构造器中声明的T形参是Integer，此时不能使用菱形语法

### 设定通配符下限

语法：`<? super Type>`,意思是？的值不能比Type的低了，至少是它爹。

### 泛型方法与方法重载

因为泛型既允许设定通配符的上限，也允许设定通配符的下限，从而允许在一个类里包含如下两个方法定义。

	public class MyUtils{
		public static <T> void copy(Collection<T> dest,Collectioon<? extends T> src){...} 
		public static <T> T copy<Collection<? super T> dest,Collection<T> src){...}
	}

但在调用`copy()`方法的时候将会出错，因为编译器无法确定到底要调用哪个`copy()`方法,将引起编译错误。

## Lambda表达式（匿名方法）

### 基本概述

只有一个抽象方法的接口被称为函数式接口（functional interface）,函数式接口可以包含多个默认方法、类方法、但只能声明一个抽象方法。

>Java8专门为函数式接口提供了`@FunctionalInterface`注解。

Lambda表达式支持将**代码块**作为方法参数，Lambda表达式允许使用更简洁的代码来创建只有一个抽象方法的接口的实例。Lambda表达式的主要作用就是代替匿名内部类的繁琐语法，Lambda表达式其实就相当于一个**匿名方法**。（其实就是函数式接口的匿名内部类实现的简化版：不需要new，不需要接口名，不需要抽象方法名，不需抽象方法返回值类型；只需要抽象方法列表，只需要具体方法实现）。

Lambda表达式由三部分组成：

1. 形参列表：允许省略形参类型；**单参数**允许省略形参括号只写形参名；**无参**需写空括号（）。
2. 箭头：->
3. 代码块：单条语句允许省略代码块的花括号；但return语句允许省略return关键字。

[Java默认utf-8,但sublime默认gbk，因此应将gbk转为utf-8](http://jingyan.baidu.com/article/fc07f98972ee0a12fee51943.html)

Lambda表达式的类型，也被称为“目标类型（target type）”，Lambda表达式的目标类型必须是函数式接口。由于Lambda表达式的结果就是被当成对象，因此程序中完全可以使用Lambda表达式进行赋值，如：

	Runable r= ()->{...}

由于Lambda表达式实现的是匿名方法，因此它只能实现特定函数式接口中的唯一方法。

### 方法引用与构造器引用

若Lambda表达式的代码块只有一条代码：

1. 省略Lambda表达式中代码块的花括号
2. 使用方法引用和构造器引用

方法引用和构造器引用可以让Lambda表达式的代码块更加简洁，二者都需要使用两个英文冒号。

+ 引用类方法：

如下：

	@FunctionalInterface
	interface Converter{
		Interfer convert(String from);
	}

的实例可以是以下两种：

	Converter converter1 = from->Integer.valueOf(from);
	等同于
	Coverter converter2 = Integer::valueOf;

+ 引用特定对象的实例方法：

如下：

    Converter comverter1 = from->"fkit.org".indexOf(from);//得出参数在字符串“fkit.ort"中的位置。
	等同于
	Converter converter2 = "fkit.org"::indexOf;//函数式接口中被实现方法的全部参数传给该方法作为参数。

+ 引用某类对象的实例方法：

如下：

	@FunctionalInterface
	interface MyTest{
		String test(String a,int b,int c);
	}

的实例可以是以下两种：

	MyTest mt = (a,b,c)->a.substring(b,c);
	等同于
	MyTest mt = String::substring;//函数式接口中被实现方法的第一个参数作为调用者，后面的参数全部传给该方法作为参数

+ 引用构造器

如下：

	@FunctionalInterface
	interface YourTest{
		JFrame win(String title);
	}
	
的实例可以是以下两种：

	YourTest yt = (String a)->new JFrame(a);
	等同于：
	YourTest yt = JFrame::new;

### Lambda表达式与匿名内部类的联系及区别

相同点：

1. 都可以访问“effectively final”的局部变量，以及外部类的成员变量（包括成员变量和类变量）；
2. 二者创建的**对象**都可以直接调用从接口中继承的**默认方法**。

不同点：

1. 匿名内部类可以为任意接口创建实例，但Lambad表达式只能为函数式接口创建实例；
2. 匿名内部类可以为抽象类甚至普通类创建实例，但Lambda表达式只能为函数式接口创建实例；
3. 匿名内部类实现的抽象方法的方法体允许调用接口中定义的默认方法，但Lambda表达式的**代码块**中不允许调用接口中定义的**默认方法**。

### 使用Lambda表达式调用Arrays的类方法

[疯狂Java讲义P220]()：

	String[] arr1 = new String[]{"java","fkava","fkit","ios","android"};
	Arrays.parallelSort(arr1,(o1,o2)->o1.length()-o2.length());//按字母长度排序
	
	long[] arr2 = new long[5];
	Arrays.paralleSetAll(arr2,operand->operand*5);

## 正则表达式

判断字符串是否以某个字符串开头：`string.startsWith("x");`

判断一个字符是否是数字：`Character.isDigit(‘x’);`

### 语法

1. 字符
	+ x:字符x。举例：‘a’表示字符a
	+ `\\`:反斜线字符
	+ \n:新行（换行）符（'\u000a'）
	+ \r:回车符（'\u000d'）
2. 字符类
	+ [abc]:a、b或c（简单类）
	+ [^abc]:任何字符，除了a、b或c(否定)
	+ [a-zA-Z]:a到z或A到Z，两头的字符包括在内（范围）
	+ [0-9]:0到9的字符都包括
3. 预定义字符类
	+ .：任何字符。就是.字符本身，怎么表示呢？\.
	+ \d:数字，[0-9]
	+ \w:单词字符：[a-zA-Z_0-9],在正则表达式里面组成单此的东西必须由这些东西组成
4. 边界匹配器
	+ ^：行的开头
	+ $行的结尾
	+ \b:单此边界
5. 数量词
	+ X？   X,一次或一次也没有（要么唯一，要么没有）
	+ X*    X,零次或多次（任意）
	+ X+    X,一次或多次（至少一次）
	+ X{n}  X,恰好n次
	+ X{n，} X,至少n次
	+ X{n，m} X，至少n次，至多m次

如：不能以0开头的5-15为QQ号的验证：`"[1-9][0-9]{4,14}"`

又等于：`"[1-9]\\d{4,14}"`