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