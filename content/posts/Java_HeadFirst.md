---
title: "Java基础学习心得--权限修饰符"
date: 2018-08-15T14:04:13+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","基础语法","面向对象"]
author: "Jason·Gan"
---

* **前言：最近开始学习了Java的基础语法，其实编程语言核心思想大体相通，语法也是有共同之处，所以语法学习起来还是比较亲切的，只是底层的原理、JVM、好多些框架才是头痛的呀，😭。**  
## 1.面向对象之---权限修饰符  
* public、private、protected、default   
* public members:  
如果我们声明了一个方法或者变量为public，那么我们可以在任何地方都可以通过对象读取到他，但是前提是相关的类声明也是public  
* private members:  
如果类声明的方法或者变量是private，那么我们只能在该类的内部使用它。抽象的方法应该对他的子类都可见，而private的方法将不可见，因此，用private和abstract结合是非法的。（abstract是用来继承的，意思是抽象的）  
* protected members:  
如果我们声明一个方法是protected，那我们可以在该package里都可以读取它，但如果要在别的package读取到该方法的话，只能是该类的子类才可以。  
* default members:  
default就是没有任何权限修饰符，这样的方法变量只能在当前package里读取，意味着default代表是package层次的权限修饰。更为直观的区别，请看下图：  
<span align="center">![](/image/HeadFirst/access.png)</span>

