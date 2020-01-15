title: java线程创建全家桶
tags:
  - Thread
categories:
  - Java
date: 2019-09-16 13:52:00
cover: true

---
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1mZTJhMWRhM2Q1NGQ3OTgxLmpwZw?x-oss-process=image/format,png )
<!-- more -->

## 继承Thread类
```
//继承Thread
public class ExtendThread extends Thread{
   //线程执行体
   @Override
   public void run() {
       //do something
       System.out.println("继承Thread创建线程");
       //无返回值
  }
}
public class ThreadCreateDemo {
   public static void main(String[] args) {
       //创建一个线程
       ExtendThread extendThread = new ExtendThread();
       //调用start方法启动线程
       extendThread.start();
        //没有返回值
  }
}
```
`使用继承Thread类的方法来创建线程类时候，多个线程之间是无法共享线程类的实例变量的。`

## 实现Runnable接口

覆写Runnable接口实现多线程可以避免单继承局限， 当子类实现Runnable接口，此时子类和Thread的代理模式（子类负责真实业务的操作，thread负责资源调度与线程创建辅助真实业务）。
```
//实现Runnable接口
public class ImplRunnable implements Runnable {
   //线程实行体
   @Override
   public void run() {
       //do something
       System.out.println("实现Runnable创建线程");
       //没有返回值
  }
}
public class ThreadCreateDemo {
   public static void main(String[] args) {
       ImplRunnable implRunnable = new ImplRunnable();
       Thread thread = new Thread(implRunnable);
       //启动线程
       thread.start();
  }
}
```
`Runnable对象仅仅作为Thread对象的target，Runnable实现类里包含的run方法仅仅作为线程的执行体，而实际的线程对象依旧是Thread实例，只是该Thread线程负责执行器target的方法。`

## 覆写Callable接口
```
//实现Callable返回值类型为Integer类型
public class ImplCallable implements Callable<Integer> {
   //该call()方法将作为线程执行体，并且有返回值
   @Override
   public Integer call() throws Exception {
       //do something
       System.out.println("实现Callable接口创建线程，返回类型为Integer类型");
       return 999;
  }
}
public class ThreadCreateDemo {
   public static void main(String[] args) throws ExecutionException, InterruptedException {
       Callable<Integer> callable = new ImplCallable();
       FutureTask<Integer> futureTask = new FutureTask<>(callable);
       Thread thread = new Thread(futureTask);
       thread.start();
       //获取返回值futureTask.get()
       System.out.println(futureTask.get());
  }
}
```
`Callable接口有泛型限制，Callable接口里的泛型形参类型与call方法返回值类型相同，而且Callable接口是函数式接口，因此可以使用Lambda表达式创建Callable对象。`

## 三种方式的对比

通过继承Thread类或者实现Runnable接口、Callable接口都可以实现多线程，不过实现Runnable接口与实现Callable接口的方式基本相同，只是Callabl接口里定义的方法返回值，可以声明抛出异常而已。因此将实现Runnable接口和实现Callable接口归为一种方式。这种方式与继承Thread方式之间的主要差别如下。

### 采用实现Runnable、Callable接口的方式创建线程的优缺点

`优点`
线程类只是实现了Runnable或者Callable接口，还可以继承其他类。这种方式下，多个线程可以共享一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好的体现了面向对象的思想。
`缺点`
编程稍微复杂一些，如果需要访问当前线程，则必须使用
Thread.currentThread()方法

### 采用继承Thread类的方式创建线程的优缺点

`缺点`
因为线程类已经继承了Thread类，Java语言是单继承的，所以就不能再继承其他父类了。

`优点`
编写简单，如果需要访问当前线程，则无需使用
Thread.currentThread()方法，直接使用this即可获取当前线程