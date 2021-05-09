---
title: "Java 注解原理解析"
date: 2019-09-02T21:16:54+08:00
draft: false
tags: ["Java","Annotation","@","Proxy"]
categories: ["Java相关"]
author: "Jason·Gan"
---  
* 前言：在学习Spring的过程中，或者其他的一些框架，都会接触到许多方便又有趣的注解，这些注解不仅没有很强的侵入性，而且让我们感受到了框架的曼妙。相对于XML的极大松耦合来说，注解处于一个中庸之道，所以了解其内部原理是有必要的。  

## 注解的定义  
* Annontation是Java5开始引入的新特征，中文名称叫注解。它提供了一种安全的类似注释的机制，用来将任何的信息或元数据（metadata）与程序元素（类、方法、成员变量等）进行关联。为程序的元素（类、方法、成员变量）加上更直观更明了的说明，这些说明信息是与程序的业务逻辑无关，并且供指定的工具或框架使用。Annontation像一种修饰符一样，应用于包、类型、构造方法、方法、成员变量、参数及本地变量的声明语句中。  
* Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。  
* 注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。  
* 「java.lang.annotation.Annotation」接口中有这么一句话：  
```
 The common interface extended by all annotation types 
```  
* 其实注解就是一个继承了Annotation这个接口的一个接口  

## 元注解  
* 元注解用于修饰注解的注解，通常用在注解的定义上,用于指定某个注解生命周期以及作用目标等信息：  
```

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
    ...
}  
```  
* Java中有以下几个元注解：  
  1. @Target：注解的作用目标
  2. @Retention：注解的生命周期
  3. @Documented：注解是否应当被包含在 JavaDoc 文档中
  4. @Inherited：是否允许子类继承该注解  
* 其中：  
  1. @Target 用于指明被修饰的注解最终可以作用的目标是谁，也就是指明，你的注解到底是用来修饰方法的？修饰类的？还是用来修饰字段属性的。
  @Target 定义如下：  
  ![](/image/Annotation/1.jpg)
  ElementType参数包括：  
  ```  

  ElementType.CONSTRUCTOR:用于描述构造器
  ElementType.FIELD:成员变量、对象、属性（包括enum实例）
  ElementType.LOCAL_VARIABLE:用于描述局部变量
  ElementType.METHOD:用于描述方法
  ElementType.PACKAGE:用于描述包
  ElementType.PARAMETER:用于描述参数
  ElementType.TYPE:用于描述类、接口(包括注解类型) 或enum声明

  ```  

  2. @Retention 用于指明当前注解的生命周期，它的基本定义如下： 
  ![](/image/Annotation/2.jpg)  
  这里的 RetentionPolicy 依然是一个枚举类型:  
  ```

  RetentionPolicy.SOURCE：当前注解编译期可见，不会写入 class 文件
  RetentionPolicy.CLASS：类加载阶段丢弃，会写入 class 文件
  RetentionPolicy.RUNTIME：永久保存，可以反射获取  
  ```  
  3. 剩下两种类型的注解我们日常用的不多，也比较简单  
## Java内置三大注解  
* @Override  
* @Deprecated  
* SuppressWarning  
* 前两个都是典型的标记式注解，仅被编译器可知，编译器在对 java 文件进行编译成字节码的过程中，一旦检测到某个方法上被修饰了该注解，就会去匹对父类中是否具有一个同样方法签名的函数，或者就会标记为过时的东西，如果不是，自然不能通过编译。  

## 注解的实现  
* 先自定义一个注解：  
![](/image/Annotation/3.jpg)  
* 虚拟机规范定义了一系列和注解相关的属性表，也就是说，无论是字段、方法或是类本身，如果被注解修饰了，就可以被编译成字节码。对于一个类或者接口来说，Class 类中提供了以下一些方法用于反射注解：  
  1. getAnnotation：返回指定的注解
  2. isAnnotationPresent：判定当前元素是否被指定注解修饰
  3. getAnnotations：返回所有的注解 
  4. getDeclaredAnnotation：返回本元素的指定注解  
  5. getDeclaredAnnotations：返回本元素的所有注解，不包含父类继承而来的  
* 注解本质上是继承了 Annotation 接口的接口，而当你通过反射，也就是我们这里的 getAnnotation 方法去获取一个注解类实例的时候，其实 JDK 是通过动态代理机制生成一个实现我们注解（接口）的代理类。  
![](/image/Annotation/4.jpg)  
运行上面程序，得到一个代理类，代理类实现接口MyAnnotation并重写其所有方法，包括value方法以及接口MyAnnotation从Annotation接口继承而来的方法。其中有一个AnnotationInvocationHandler类，nnotationInvocationHandler 是 Java 中专门用于处理注解的 Handler  
![](/image/Annotation/5.jpg)  
这里有一个 memberValues，它是一个 Map 键值对，键是我们注解属性名称，值就是该属性当初被赋上的值  
![](/image/Annotation/6.png)  
![](/image/Annotation/7.png)  

代理类代理了MyAnnotation接口中所有的方法，所以对于代理类中任何方法的调用都会被转到这里来。var2指向被调用的方法实例，而这里首先用变量var4获取该方法的简明名称，接着switch结构判断当前的调用方法是谁，如果是 Annotation中的四大方法,将var7赋上特定的值。
如果当前调用的方法是toString,equals，hashCode，annotationType 的话，AnnotationInvocationHandler 实例中已经预定义好了这些方法的实现，直接调用即可。

那么假如 var7 没有匹配上这四种方法，说明当前的方法调用的是自定义注解字节声明的方法，例如我们MyAnnotation注解的value方法。这种情况下，将从我们的注解map中获取这个注解属性对应的值。  

## 总结

* 1 我们通过键值对的形式可以为注解属性赋值，像这样：@MyAnnotation(value = "Hello Jason")。
* 2 你用注解修饰某个元素，编译器将在编译期扫描每个类或者方法上的注解，会做一个基本的检查，你的这个注解是否允许作用在当前位置，最后会将注解信息写入元素的属性表。
* 3 当你进行反射的时候，虚拟机将所有生命周期在 RUNTIME 的注解取出来放到一个 map 中，并创建一个 AnnotationInvocationHandler 实例，把这个 map 传递给它。
* 4 虚拟机将采用 JDK 动态代理机制生成一个目标注解的代理类，并初始化好处理器。
* 5 一个注解的实例就创建出来了，它本质上就是一个代理类，你应当去理解好 AnnotationInvocationHandler 中 invoke 方法的实现逻辑，这是核心。一句话概括就是，通过方法名返回注解属性值。






