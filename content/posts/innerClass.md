---
title: "内嵌类、内嵌静态类"
date: 2019-01-24T17:51:37+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","基础语法","面向对象","内置类"]
author: "Jason·Gan"
---
* 前言： Java允许在类的内部再嵌入一个类，这样的写法成为嵌入类。内部类可以声明public、protected、private等访问限制，可以声明为abstract的供其他内部类或外部类继承与扩展，或者声明为static、final的，也可以实现特定的接口。外部类按常规的类访问方式使用内部 类，唯一的差别是外部类可以访问内部类的所有方法与属性，包括私有方法与属性。  

```
class OuterClass {
    ...
    class NestedClass {
        ...
    }
}  
```  

## 嵌入类  
* 静态嵌入类  
* 普通内部类  

```  
class OuterClass {
    ...
    static class StaticNestedClass {
        ...
    }
    class InnerClass {
        ...
    }
}
``` 
* 嵌入类是包含他的内部类的成员，非静态类能够访问包含类的其他非静态成员；而静态嵌入类不能够访问包含类的其他成员。 
 ## 使用时机  
* 只在一个地方使用这个类时，这样内嵌一个类可以让代码更简洁  
* 增加封装性，如果有两个类A和B，而B需要访问A类中的成员方法，如果不是B要访问的话，这个A中方法就可以被声明为private；此时，如果把B内嵌到A中，既可以把A的方法私有化，B又可以访问到这个方法。  
* 增加可读性和可维护性  
## 静态内嵌类  
* 和类的方法、成员一样，内嵌类也和包含它的外部类有一样的联系。静态内嵌类不能够直接访问类的其他非静态成员，只能通过创建对象来访问，但是可以直接访问静态代码块。  
![](/image/incline/1.jpg)  

* 通过外部类名来访问内嵌类：OuterClass.StaticNestedClass,例如通过内嵌类来创建对象：OuterClass.StaticNestedClass nestedObject =new OuterClass.StaticNestedClass();  
## 内嵌普通类  
* 和外部类的其他方法、成员一样，内嵌类和外部类的实例化对象有关，可以直接访问类的其他成员方法，因为内嵌普通类可以直接实例化来调用  
* 内嵌类和外部类的对象相关，内嵌类不能定义静态成员  
* 实例化内部类，不能直接实例化，需要先实例化外部类，再通过外部类对象进行实例化内部类：  
![](/image/incline/2.jpg)  
* 实例化静态内部类和普通内部类方法的对比：  
![](/image/incline/3.jpg)  
## 类的遮蔽  
* 如果在一个特定的范围内声明一个变量，而在另一个地方也声明了同名的变量，那么这样的声明会屏蔽了外面的变量。你不能仅仅通过变量名去定位这些同名的变量。举个例子：  
```
public class ShadowTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }

    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}  
```  
* 上面例子的输出是：  
```  
x = 23
this.x = 1
ShadowTest.this.x = 0
```  
* 这个例子有三个同名的变量x，methodInFirstLevel方法的形参一个，内部类FirstLevel一个，外部类ShadowTest一个。因此直接调用x，指向的是方法的变量；this.x指向的时内部类的变量，ShadowTest.this.x指向的是外部类的变量，也就是```外部类名.this.x```   








