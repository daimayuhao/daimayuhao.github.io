---
layout:     post
title:      synchronized关键字（2）
subtitle:   
date:       2017-05-14
author:     lessyh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags: 多线程

---
## 内容思维导图
![](https://ws1.sinaimg.cn/large/006rNwoDgy1fpmwg8mb6qj31480c7wf0.jpg)

## 1 synchronized 方法的缺点
如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里获取锁的线程释放锁只会有两种情况：

 1. 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；
 2. 线程执行发生异常，此时JVM会让线程自动释放锁

如果这个获取锁的线程由于要等待IO或者其他原因（比如调用sleep方法）被阻塞了，但是又没有释放锁，其他线程便只能等待！效率随之大大降低。

##  2(3)synchronized（this）同步代码块使用

**synchronized(同一个数据){
 可能会发生线程冲突问题
}**

代码:
```
private static Object oj = new Object();   	
public void sale() {
		// 前提 多线程进行使用、多个线程只能拿到一把锁。
		// 保证只能让一个线程 在执行 缺点效率降低
		 synchronized (oj) {
		if (count > 0) {
			System.out.println(Thread.currentThread().getName() + ",出售第" + (100 - count + 1) + "票");
			count--;
		}
		}
	}
```

## 4 synchronized代码块间的同步性
当一个对象访问synchronized(this)代码块时，其他线程对同一个对象中所有其他synchronized(this)代码块代码块的访问将被阻塞，这说明synchronized(this)代码块使用的“对象监视器”是一个。 
也就是说和synchronized方法一样，synchronized(this)代码块也是锁定当前对象的。

两个结论:

 1. 其他线程执行对象中synchronized同步方法和synchronized(this)代码块时呈现同步效果;
 2. 如果两个线程使用了同一个“对象监视器”,运行结果同步，否则不同步.

## 5 静态同步synchronized方法与synchronized(class)代码块

方法上加上static关键字，使用synchronized 关键字修饰 或者使用类.class文件。静态的同步函数使用的锁是  该函数所属字节码文件对象 可以用 getClass方法获取，也可以用当前  类名.class 表示。

```
synchronized (ThreadTrain.class) {
			System.out.println(Thread.currentThread().getName() + ",出售 第" + (100 - trainCount + 1) + "张票.");
			trainCount--;
			try {
				Thread.sleep(100);
			} catch (Exception e) {
			}
}
```
* synchronized 修饰方法使用锁是当前this锁。
* synchronized 修饰静态方法使用锁是当前类的字节码文件

## 6 数据类型String的常量池属性

```
    String s1 = "a";
    String s2="a";
    System.out.println(s1==s2);//true
```
 
字符串常量池中的字符串只存在一份！ 即执行完第一行代码后，常量池中已存在 “a”，那么s2不会在常量池中申请新的空间，而是直接把已存在的字符串内存地址返回给s2。

因为数据类型String的常量池属性，所以synchronized(string)在使用时某些情况下会出现一些问题，比如两个线程运行 
synchronized(“abc”)｛ 
｝和 
synchronized(“abc”)｛ 
｝修饰的方法时，这两个线程就会持有相同的锁，导致某一时刻只有一个线程能运行。所以尽量不要使用synchronized(string)而使用synchronized(object)
 
 
参考： 
**《Think in Java》**











 

