---
layout: post
title:  "Java Annotation(注解)"
date:   2017-02-05
categories: 知识点
tags: 注解
---

* content
{:toc}

本文主要介绍了Java的Annotation支持，通过使用Annotation可以为程序提供一些元数据，这些元数据可以在编译、运行时被读取，从而提供更多额外的处理信息。





### 什么是注解

Annotation(注解)是JDK5.0及以后版本引入的。它可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。从某些方面看，annotation就像修饰符一样被使用，并应用于包、类 型、构造方法、方法、成员变量、参数、本地变量的声明中。这些信息被存储在Annotation的“name=value”结构对中。

### 什么是元数据

元数据的功能作用有很多，比如：你可能用过Javadoc的注释自动生成文档。这就是元数据功能的一种。总的来说，元数据可以用来创建文档，跟踪代码的依赖性，执行编译时格式检查，代替已有的配置文件。如果要对于元数据的作用进行分类，目前还没有明确的定义，不过我们可以根据它所起的作用，大致可分为三类： 

1. 编写文档：通过代码里标识的元数据生成文档
2. 代码分析：通过代码里标识的元数据对代码进行分析
3. 编译检查：通过代码里标识的元数据让编译器能实现基本的编译检查

### 注解的分类

根据注解的**参数个数**可分为：

1. 标记注解：仅利用自身的存在与否来提供信息。（@Override 该注解只能修饰方法,@Inherited）
2. 单值注解
3. 完整注解

根据注解的**用途分类**可分为：

1. Java基本注解（JDK定义）
	- @Override
	- @Deprecated
	- @SuppressWarnings
2. 元注解（负责注解其他注解的定义的注解）
	- @Retention
	- @Target
	- @Documented
	- @Inherited

### 自定义Annotation

自定义Annotation的语法为：

	modifiers @interface AnnotationName
	{
	element declaration1
	element declaration2
	. . .
	}

modifiers指：public，protected，private或者默认值（什么也没有）。一个元素的声明（element declaration）指：`type elementName();`或者`type elementName() default value;`

例如下面代码定义了一个Annotation：

	public @interface BugReport
	{
	String assignedTo() default "[none]";
	int severity() = 0;
	}

而可以通过以下的方式来声明Annotation：

	@AnnotationName(elementName1=value1, elementName2=value2, . . .)

元数声明的顺序没有关系，有默认值的元素的声明可以不列在初始化表中，此时他们使用默认值。例如，根据上述定义如下的三个Annotation的声明是等价的：

+ @BugReport(assignedTo="Harry", severity=0)
+ @BugReport(severity=0， assignedTo="Harry")
+ @BugReport(assignedTo="Harry")

那些只有一个元素，且元素名字叫value的Annotation可以使用`@AnnotationName（“somevalue”）`的方式声明。

如果Annotation的元素是数组，则可以做如下声明：

    @BugReport(. . ., reportedBy={"Harry", "Carl"})

如果数组中只有一个元素时可以做如下声明：

    @BugReport(. . ., reportedBy="Joe") // OK, same as {"Joe"}

如果Annotation元素类型为Annotation时可以做如下声明：

    @BugReport(testCase=@TestCase(id="3352627"), . . .)

### Annotation应用实例

使用Annotation的一个例子就是建立一个简单的用户输入验证框架，使用这个框架最终用户可以以如下的方式来定义字段的校验属性：

	@ValidateRequired
	@ValidateEmail
	public void setEmail(String email) {
		this.email = email;
	}

	@ValidateRequired
	@ValidateLength(min=6,max=12)
	public void setPassword(String password) {
		this.password = password;
	}

以上的代码说明，email字段是必须的，且必须满足email的校验要求，同时password字段也是必须的，且长度必须在6~12之间。有了这些定义之后我们能够使用如下的代码达到验证的效果：

	Validator.validate(loginBean, "email", "yourname@onjava.com"); //pass
	Validator.validate(loginBean, "password", ""); // failure

要能够达到上述的要求，我们必须定义一些Annotation，以下代码是ValidateLength和ValidateExpr的声明：

	package annotations.validates;

	//Example @ValidateLength(min=6,max=8)
	public @interface ValidateLength {
	    int min() default 0;
	    int max() default Integer.MAX_VALUE;
	}

	//Example @ValidateExpr("^(\\w){0,2}$");
	public @interface ValidateExpr {
		String value();
	}

具体开发的过程中我们会遇到一些问题，这主要由于两方面的原因产生：

1. Annotation内部不能定义方法，只能定义一些状态;
2. Annotation不允许使用继承（extends或者implements），这意味着我们不能在反射的过程中使用instance of这样的功能。

为了识别出于我们定义的校验相关的Annotation我们定义了一个如下的Annotation：

	package annotations.validates;
	import java.lang.annotation.*;

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.ANNOTATION_TYPE)
	public @interface Validate {
	    //Class value();
	    Class value();
	}

正如你所看到的Validate可以驻留在JVM内部，即它可以在运行时通过反射的方式使用。同时它必须用来标记其他的Annotation，同时他有一个Class类型的value元素，这个类型必须从ValidateHandler继承而来，主要用来处理具体的验证逻辑。在此设计之下看看我们如何声明一个ValidateExpr的Annotation对象：

	package annotations.validates;
	import java.lang.annotation.*;

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	@Validate(ValidateExprHandler.class)
	public @interface ValidateExpr {
	    String value();
	}

ValidateExpr的前两个Annotation不用多讲，主要说说`@Validate(ValidateExprHandler.class)`的含义，这样解决了我们前边提到的两个问题，第一、我们可以看看一个Annotation是否有Validate类型的Annotation来确定这个Annotation是不是我们校验框架内部使用的Annotation。同时我们也提供了一个具体的类ValidateExprHandler来处理校验逻辑。接下来我们看看ValidateExprHandler的实现:

	package annotations.validates;
	import java.lang.annotation.Annotation;
	import javax.xml.bind.ValidationException;

	//定义了一个ValidateHandler接口，
	//这个接口有一个Annotation类型的模版参数
	public interface ValidateHandler {
	    public void validate(T settings, Object value) throws ValidationException;
	    public Class getSettingsType();
	}

以下代码是一个ValidateHandler的实例，用来处理正则表达式的验证

	package annotations.validates;
	import java.util.regex.Pattern;
	import javax.xml.bind.ValidationException;
	//一个ValidateHandler的实例，用来处理正则表达式的验证，
	//其中的Anotation类型的参数为ValidateExpr
	public class ValidateExprHandler implements ValidateHandler {
	    public void validate(ValidateExpr settings, Object value)
	            throws ValidationException {
	        // TODO Auto-generated method stub
	        String i = (value != null) ? value.toString() : "";
	        if (!Pattern.matches(settings.value(), i)) {
	            throw new ValidationException(i + " does not match the pattern "
	                    + settings.value());
	        }
	    }
	    public Class getSettingsType() {
	        // TODO Auto-generated method stub
	        return ValidateExprHandler.class;
	    }
	}

**说明**：

1. 我们定义了一个Annotation（Validate）来标记我们所有的校验用Annotation，同时每一个具体的校验用的Annotation（ValidateExpr）都必须提供一个用来具体处理校验逻辑的类（ValidateExprHandler）。
2. Annotation不允许继承，所以我们没有办法适用instance of的方法来识别一个校验框架使用的Annotation，但是通过对我们使用的校验用的Annotation添加Annotation（Validate）我们同样可以达到以上的目的。
3. ValidateHandler接口允许一个校验用的Annotation将具体的校验功能已代理的方式让其它的类来完成。我们可以使用如下的方式来处理校验的具体过程：

在JDK1.5中Method实现了AnnotatedElement接口，我们可以使用AnnotatedElement来做处理操作

	// 对一个方法和将要调用的参数值进行校验
    public static void validate(AnnotatedElement element, Object value) {
        Validate v;
        ValidateHandler vh;
        Annotation a;
        // 从该方法上返回所有的Annotation
        Annotation[] annotations = element.getAnnotations();
        for (int i = 0; i < annotations.length; i++) {
            // 如果该Annotation有类型为Validate的Annotation，则说明这是我们校验
            // 框架所使用的Annotation。
            v = annotations[i].annotationType().getAnnotation(Validate.class);
            if (v != null) {
                try {
                    // 使用Annotation中定义的ValidateHandler类来生成ValidateHandler的实例
                    vh = v.value().newInstance();
                    // 使用创建的ValidateHandler来做校验操作。
                    // 校验过程中可以抛出ValidationException
                    vh.validate(annotations[i], value);
                } catch (Exception e) {
                    // TODO: handle exception
                    e.printStackTrace();
                }
            }
        }

## 参考

[一小时搞明白自定义注解（Annotation）](http://blog.csdn.net/u013045971/article/details/53433874)

[Java之metadata(元数据)详解](http://blog.csdn.net/u013045971/article/details/53433874)

[疯狂Java讲义 P642]()