---
title: "volatile关键字相关"
date: 2019-05-01T19:21:50+08:00
draft: false
categories: ["Java相关"]
tags: ["Java","多线程","线程安全","并发编程"]
author: "Jason·Gan"
---

* Java语言提供了一种稍弱的同步机制，即volatile变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为volatile类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile变量不会被缓存到寄存器或对其他处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值。  

## volatile特性  
* 可见性，当一条线程对volatile变量进行修改时，其他线程能立即知道修改的值，即当读取一个volatile变量时总是返回最近一次写入的值。  
* 写入volatile变量相当于退出同步代码块，而读取volatile变量相当于进入同步代码块。  
* 仅当volatile变量能简化代码的实现以及对同步策略的验证时，才应该使用他们。读取volatile变量的开销只比读取非volatile变量的开销略高一些。  
* volatile变量只能确保可见性  

## 使用volatile变量的前提  
* 对变量的写入操作不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值
* 该变量不会与其他状态变量一起纳入不变性条件中
* 在访问变量时不需要加锁 

## volatile变量可见性  
* volatile的可见性正是基于happend -before(先行发生)关系实现的。
* happend-before：java内存模型有八条可以保证happend-before的规则（详见《深入理解Java虚拟机》P376），如果两个操作之间的关系无法从这八条规则中推导出来的话，它们就没有顺序保障，虚拟机就可以对它们随意地进行重排序。其中就包含”volatile变量规则“：对一个volatile变量的写操作先行发生于后面对这个变量的读操作，此规则保证虚拟机不会对volatile读/写操作进行重排序。
* 通过一个例子来了解vloative的可见性：  
```

public class VolatileTest extends  Thread{
    private boolean isRunning = true;
    public boolean isRunning(){
        return isRunning;
    }
    public void setRunning(boolean isRunning){
        this.isRunning= isRunning;
    }
    public void run(){
        System.out.println("进入了run...............");
        while (isRunning){}
        System.out.println("isRunning的值被修改为为false,线程将被停止了");
    }
    public static void main(String[] args) throws InterruptedException {
        VolatileTest volatileThread = new VolatileTest();
        volatileThread.start();
        Thread.sleep(1000);
        volatileThread.setRunning(false);   //停止线程
    }
}
```  
* 输出：```进入了run............... ```  
* 发现并没有输出”isRunning的值被修改为为false,线程将被停止了”这一句，说明通过setRunning来修改isRunning的值对于该程序是不可见的，也就是说程序不知道自己的值被修改了，为什么？
* 原因：Java内存模型(JMM)规定了所有的变量都存储在主内存中，主内存中的变量为共享变量，而每条线程都有自己的工作内存，线程的工作内存保存了从主内存拷贝的变量，所有对变量的操作都在自己的工作内存中进行，完成后再刷新到主内存中，回到例1，第18行号的代码中主线程(线程main)虽然对isRunning的变量进行了修改且有刷新回主内存中（《深入理解java虚拟机》中关于主内存与工作内存的交互协议提到变量在工作内存中改变后必须将该变化同步回主内存），但volatileThread线程读的仍是自己工作内存的旧值导致出现多线程的可见性问题，解决办法就是给isRunning变量加上volatile关键字。
* 当变量被声明成volatile类型后，线程对该变量进行修改后会立即刷新回主内存，而其他线程读取该变量时会先将自己工作内存中的变量置为无效，再从主内存重新读取变量到自己的工作内存，这样就避免发生线程可见性问题。  

## volatile变量不具备原子性  
* 对复合操作存在线程不安全的问题  
* 例如：  
```

public class VolatileTest1{
    private volatile  int value;  //将value变量声明成volatile类型
    public  void increment(){
         value ++;  
         System.out.println(value);  
    }
    public static void main(String[] args) {
        final VolatileTest1 volatileTest1 = new VolatileTest1();
        for(int i = 0; i < 10; i ++){
            new Thread(new Runnable() {
                public void run() {
                     volatileTest1.increment();
                }
            }).start();
        }
    }
}
```   
* 输出：  
```

1
6
5
4
2
3
8
7
9
10
```
* 当线程1在步骤2对value进行计算时，刚好其他线程也对value进行了修改，这时线程1返回的值就不是我们期望的值了，于是出现线程安全问题，所以volatile不能保证复合操作具有原子性；解决办法就是给increment方法加锁(lock/synchronized)或将变量声明为原子类类型。  

## Synchronized与volatile区别   

1.volatile只能修饰变量，而synchronized可以修改变量，方法以及代码块
 
2.volatile在多线程中不会存在阻塞问题，synchronized会存在阻塞问题
 
3.volatile能保证数据的可见性，但不能完全保证数据的原子性，synchronized即保证了数据的可见性也保证了原子性
 
4.volatile解决的是变量在多个线程之间的可见性，而sychroized解决的是多个线程之间访问资源的同步性


