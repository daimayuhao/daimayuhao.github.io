---
layout:     post
title:      synchronized关键字（1）
subtitle:   
date:       2017-05-13
author:     lessyh
header-img: img/post-bg-github-cup.jpg
catalog: true
tags: 多线程

---
## 内容思维导图
![](https://user-gold-cdn.xitu.io/2018/8/4/16504e245ceb3ea9?w=1028&h=490&f=jpeg&s=203811)

## 1 简介
**synchronized关键字**，代表这个方法加锁,相当于不管哪一个线程（例如线程A），运行到这个方法时,都要检查有没有其它线程B（或者C、D等）正在用这个方法(或者该类的其他同步方法)，有的话要等正在使用**synchronized**方法的线程B（或者C、D）运行完这个方法后再运行此线程A,没有的话,锁定调用者,然后直接运行。它包括两种用法：**synchronized** 方法和 **synchronized** 块。

## 2 变量安全性

当多个线程同时共享，**同一个全局变量或静态变量**，做写的操作时，可能会发生数据冲突问题，也就是线程安全问题。但是做读操作是不会发生数据冲突问题。

## 3 多个对象多个锁
Demo

HasSelfPrivateNum类：

```
public class HasSelfPrivateNum {
    private int num=0;
   synchronized  public void addI(String username){
        try{
            if(username.equals("a")){
                num=100;
                System.out.println("a set over!");
                Thread.sleep(2000);
            }else{
                num=200;
                System.out.println("b set over!");
            }
            System.out.println(username+" num="+num);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
```
线程类ThreadA和ThreadB:

```
public class ThreadA extends Thread {

    private HasSelfPrivateNum numRef;

    public ThreadA(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("a");
    }

}
```

```
public class ThreadB extends Thread {

    private HasSelfPrivateNum numRef;

    public ThreadB(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("b");
    }

}
```

Run类，执行Main方法：

```
public class Run {
    public static void main(String[] args){
        HasSelfPrivateNum numRef1=new HasSelfPrivateNum();
        HasSelfPrivateNum numRef2=new HasSelfPrivateNum();
        ThreadA athread=new ThreadA(numRef1);
        athread.start();
        ThreadB bthread=new ThreadB(numRef2);
        bthread.start();
    }
}
```

运行结果:
```
a set over!
b set over!
b num=200
a num=100
```

例子是两个线程分别访问同一个类的两个不同实例的相同名称的同步方法，效果是以异步的方式运行的。原因是：示例创建了2个业务对象，在系统中产生了2个锁，所以运行结果是异步的，打印的效果是先打印了b，然后打印了a。

HasSelfPrivateNum.java中使用了synchronized关键字，但打印的顺序却不是同步的是交叉的，为什么？

关键字 synchronized取得的锁是对象锁，而不是把一段代码或方法（函数）当做锁，如果多个线程访问的是多个对象，则JVM会创建多个锁，互不影响。
如果在静态方法上加synchronized关键字，表示锁定类级别的锁，独占class类，这时候多个线程访问的是相同的锁。

## 4 synchronized方法与对象锁

首先先说明方法锁与对象锁所说的是一个东西，只有方法锁或对象锁和类锁两种。多个线程访问同一个对象中的非synchronized类型时，会异步调用非synchronized类型方法，解决方案为在这个方法上加synchronized关键字即可。

java的对象锁和类锁在锁的概念上基本上和内置锁是一致的，但是，两个锁实际是有很大的区别的，对象锁是用于对象实例方法，或者一个对象实例上的，类锁是用于类的静态方法或者一个类的class对象上的。我们知道，类的对象实例可以有很多个，但是每个类只有一个class对象，所以不同对象实例的对象锁是互不干扰的，但是每个类只有一个类锁。但是有一点必须注意的是，其实类锁只是一个概念上的东西，并不是真实存在的，它只是用来帮助我们理解锁定实例方法和静态方法的区别的

## 5 脏读

当多个线程同时共享，同一个全局变量或静态变量，做写的操作时，可能会发生数据冲突问题，也就是线程安全问题。但是做读操作是不会发生数据冲突问题。发生脏读的情况实在读取实例变量时，此值已经被其他线程更改过。

`PublicVar.java`

```
public class PublicVar {

    public String username = "A";
    public String password = "AA";

    synchronized public void setValue(String username, String password) {
        try {
            this.username = username;
            Thread.sleep(5000);
            this.password = password;

            System.out.println("setValue method thread name="
                    + Thread.currentThread().getName() + " username="
                    + username + " password=" + password);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
     //该方法前加上synchronized关键字就同步了
     public void getValue() {
        System.out.println("getValue method thread name="
                + Thread.currentThread().getName() + " username=" + username
                + " password=" + password);
    }
}
```

`ThreadA.java`

```
public class ThreadA extends Thread {

    private PublicVar publicVar;

    public ThreadA(PublicVar publicVar) {
        super();
        this.publicVar = publicVar;
    }

    @Override
    public void run() {
        super.run();
        publicVar.setValue("B", "BB");
    }
}

```

`Test.java`

```
public class Test {

    public static void main(String[] args) {
        try {
            PublicVar publicVarRef = new PublicVar();
            ThreadA thread = new ThreadA(publicVarRef);
            thread.start();

            Thread.sleep(200);//打印结果受此值大小影响

            publicVarRef.getValue();
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }
}
```

运行结果

![](https://user-gold-cdn.xitu.io/2018/3/21/1624911ac7c86cc3?w=679&h=58&f=jpeg&s=57359)

解决方法：getValue（）方法加上synchronized关键字即可

![](https://user-gold-cdn.xitu.io/2018/3/21/162491341fe4d47c?w=712&h=66&f=jpeg&s=12218)

# 6 synchronized锁重入
“可重入锁”概念是：自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。


# 7 同步不具有继承性
如果父类有一个带synchronized关键字的方法，子类继承并重写了这个方法。 
但是同步不能继承，所以还是需要在子类方法中添加synchronized关键字。




参考： 
**《Java核心技术》**











 

