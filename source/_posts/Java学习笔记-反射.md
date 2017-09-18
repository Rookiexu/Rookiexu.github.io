layout: '[layout]'
description: Java反射是Java语言的一个很重要的特征，它使得Java具有了“动态性”。在Java运行时环境中，对于一个类，能否知道这个类有哪些属性和方法？对于任意一个对象，能否调用它的任意一个方法？答案是肯定的。这种动态获取类的信息以及动态调用对象的方法的功能来自于Java语言的反射(Reflection)机制。
title: Java学习笔记-反射
date: 2016-4-27 17:18:32  
tags: [Java]
---

# 理论介绍 #
首先用文字谈谈我对于”Java反射机制”的理论介绍。

Java反射是Java语言的一个很重要的特征，它使得Java具有了“动态性”。

在Java运行时环境中，对于一个类，能否知道这个类有哪些属性和方法？对于任意一个对象，能否调用它的任意一个方法？答案是肯定的。这种动态获取类的信息以及动态调用对象的方法的功能来自于Java语言的反射(Reflection)机制。

Java反射机制主要提供了以下功能:

1. 在运行时判断任意一个对象所属的类。
2. 在运行时构造任意一个类的对象。
3. 在运行时判断任意一个类所具有的成员变量和方法。
4. 在运行时调用任意一个对象的方法。

Reflection是Java被视为动态(或准动态)语言的一个关键性质。这个机制允许程序在运行时透过Reflection APIs取得任何一个已知名称的class的内部信息，包括modifiers(诸如public,static等等)、superclass(例如Object)、实现之interfaces(例如Serializable)，也包括fields和methods的所有信息，并可于运行时改变fields内容或调用methods。

一般而言，开发者社群说到动态语言，大致认同的一个定义是:“程序运行时，允许改变程序结构或变凉类型，这种语言成为动态语言”。从这个观点看，Java就不是动态语言。

尽管在这样的定义与分类下Java不是动态语言，它却有着一个非常突出的动态相关机制:Reflection。这个字的意思是“反射、映像、倒影”，用在Java身上指的是我们可以于运行时加载、探知、使用编译器件完全未知的classes。换句话说，Java程序可以加在一个运行时才得知名称的class，获悉其完整构造(但不包括methods定义)，并生成其对象实体、或对其fields设值、或唤起其methods。这种“看透class”的能力(the ability of the program to examine itselt)被称为introspection(内省、内观、反省)。Reflection和introspection是常被并提的两个术语。

在JDK中，主要由以下类来实现Java反射机制，这些类都位于`java.lang.reflet`包中:

**Class类：代表一个类。  
field类：代表类的成员变量(成员变量也被称为类的属性)  
Method类：代表类的方法。   
Constructor类：代表类的构造方法。  
Array类：提供了动态创建数组，以及访问数组的元素的静态方法。**
# 获得Class对象 #

Java中每个类被加载之后，系统就会为该类声称一个对应的Class对象，通过该Class对象就可以访问到JVM中的这个类。在Java程序中获得Class对象通常有如下三种方式。


- 1.使用Class类的forName(String clazzName)静态方法。该方法需要传入字符串参数，该字符串参数的值是某个类的全限定类名(必须包括完整包名)。

## 代码示例： ##

    Class clazz=Class.forName(“demo.Person”);

”Demo”代表包名，”Person”代表类名。

- 2.调用某个类的class属性来获取该类对应的Class对象，例如，`Person.class`将会返回`Person`类对应的Class对象。

## 代码示例： ##

    Class clazz=Person.class;

- 3.调用某个对象的getClass()方法，该方法是java.lang.Object类中的一个方法，所以所有的Java对象都可以调用该方法，该方法将会返回该对象所属类对应的Class对象。

## 代码示例: ##

	Person person=new Person();
	Class clazz=person.getClass();

Class对象可以获得该类里的方法(由Method对象表示)、构造器(由Constructor对象表示)、成员变量(由Field)对象表示，这三个类都位于java.lang.reflect包下并实现`java.lang.reflect.Member`接口。程序可以通过Method对象来执行对应的方法，通过Constructor对象来调用对应的构造器创建实例，能通过Field对象直接访问并修改对象的成员变量值。

# 创建实例对象 #

通过反射来生成实例对象有如下两种方式。

1.使用Class对象的newInstance()方法来创建该Class对应类的实例，这种方式要求该Class对象的对应类有默认构造器，而执行newInstance()时实际上是利用默认构造器来创建该类的实例。

## 代码示例: ##

	Class clazz=Person.class;
	Object obj=clazz.newInstance();

2.先使用Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建该Class对象对应类的实例。通过这种方式可以选择使用指定的构造器来创建实例。

## 代码示例: ##

	Class clazz=Person.class;
	Constructor constructor=clazz.getConstructor(String.class);
	Object obj=constructor.newInstance();



理论说了这么多，接下来给个很全的案例大家看，毕竟”Talk is cheap,show me your code”。   最终掌握还是要多使用在以后的实际中。
