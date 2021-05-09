---
title: "String,StringBuffer,StringBuilder相关"
date: 2018-09-28T10:39:50+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","基础语法","面向对象","字符串"]
author: "Jason·Gan"
---
* String 字符串常量  
* StringBuffer 字符串变量(线程安全)
* StringBuilder 字符串变量(线程非安全)  
* String类型和StringBuffer主要的性能区别就是在于String是不可变对象，是一个常量，每次对String对象进行改变的时候等同于生成一个新的String对象，再将指针指向新的String对象，所以经常改变内容的字符串最好不要用String，因为每次产生新的对象都会产生时间开销，对性能产生影响，特别是当内存中的无引用的对象增加后，JVM的GC就会开始工作，从而性能变差。  
* StringBuffer是一个变量，每次操作都是在变量的基础上进行，而不是new新的对象，性能上比String要好，这种情况下比较推荐使用StringBuffer。但是在某些特殊情况下，会有一些看似不符合这个规则的现象出现，例：  
  * String var1="hello"+"world";
  * StringBuffer var2=new StringBuffer("hello").append("world");
  * 这种情况是第一种比较快的，但是这并不是StringBuffer的速度不行，而是JVM把第一种表达式看成了：String var1="hello world";这是在栈上定义一个局部变量，当然快。  
  * 但是如果是：String var1="hello";  
  * String var3="world";
  * String var4=var1+var3;
  * 这样的话就明显比StringBuffer慢了  

## StringBuffer  
* 可将StringBuffer安全地用于多个线程。可以在必要时对这些方法进行同步，因此任意特定实例上的所有操作就好像是以串行顺序发生的，该顺序与所涉及的每个线程进行的方法调用顺序一致。
* StringBuffer上的主要操作是 append 和 insert 方法，可重载这些方法，以接受任意类型的数据。每个方法都能有效地将给定的数据转换成字符串，然后将该字符串的字符追加或插入到字符串缓冲区中。append 方法始终将这些字符添加到缓冲区的末端；而 insert 方法则在指定的点添加字符。
* 例如，如果 z 引用一个当前内容是“start”的字符串缓冲区对象，则此方法调用 z.append("le") 会使字符串缓冲区包含“startle”，而 z.insert(4, "le") 将更改字符串缓冲区，使之包含“starlet”。  

## StringBuilder  
* java.lang.StringBuilder一个可变的字符序列是5.0新增的。此类提供一个与 StringBuffer 兼容的 API，但不保证同步。
* 该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。两者的方法基本相同。只是StringBuffer的一个简化版，性能上优于StringBuffer。

