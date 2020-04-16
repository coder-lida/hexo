title: JVM&GC(一)java引用
tags:
  - JVM
categories:
  - Dev
  - Java
date: 2020-04-16 17:39:00
cover: true
---
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/jvm.jpg)
<!-- more -->
## 前言
Java中的引用有点像C++中的指针，通过引用可以对堆中的对象进行操作。在Java程序中最常见的引用类型是`强引用`，也是默认的引用类型。当在Java语言中使用New操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。

## Jva中的引用
Java中提供了四个级别的引用，强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）。
### 强引用
在一个线程内，无需引用直接可以使用的对象，除非引用不存在了，否则强引用不会被GC清理。JVM即使抛出OOM异常，也不会回收强引用所指向的对象。强引用可能导致内存泄漏问。
```
        String str = "hello";
        String str1 = str;
        System.out.println(str==str1);
```
str变量将被分配到栈内，而“hello”对象则被分配在java堆中。局部变量str指向“hello”实例所在的堆空间，通过str可以操作该实例。此时str就是该实例的引用。
`str1 = str`此时，str所指向的对象也被str1所指向，同时会在局部栈空间上分配空间存放str1变量。
对引用使用`==`比较的是两个引用所指向的堆空间的地址是否相同。

### 软引用
软引用是除了强引用外，最强的引用类型。用来描述一些还有用但是并非必须的对象，在Java中用java.lang.ref.SoftReference类来表示。对于软引用关联着的对象，只有在内存不足的时候JVM才会回收该对象。因此，这一点可以很好地用来解决OOM的问题，并且这个特性很适合用来实现缓存：比如网页缓存、图片缓存等。
```
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;
sf.get();//有时候会返回null
//sf是对obj的一个软引用，通过sf.get()方法可以取到这个对象，当这个对象被标记为需要回收的对象时，则返回null；
```
SoftReference的特点是它的一个实例保存对一个Java对象的软引用，该软引用的存在不妨碍垃圾收集线程对该Java对象的回收。也就是说，一旦SoftReference保存了对一个Java对象的软引用后，在垃圾线程对 这个Java对象回收前，SoftReference类所提供的get()方法返回Java对象的强引用。一旦垃圾线程回收该Java对象之后，get()方法将返回null。

软引用主要用户实现类似缓存的功能，在内存足够的情况下直接通过软引用取值，无需从繁忙的真实来源查询数据，提升速度；当内存不足时，自动删除这部分缓存数据，从真正的来源查询这些数据。使用软引用能防止内存泄露，增强程序的健壮性。软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。也就是说，ReferenceQueue中保存的对象是Reference对象，而且是已经失去了它所软引用的对象的Reference对象。当调用它的poll()方法的时候，如果这个队列中不是空队列，那么将返回队列前面的那个Reference对象。在任何时候，都可以调用ReferenceQueue的poll()方法来检查是否有它所关心的非强可及对象被回收。如果队列为空，将返回一个null,否则该方法返回队列中前面的一个Reference对象。利用这个方法，可以检查哪个SoftReference所软引用的对象已经被回收，于是可以把这些失去所软引用的对象的SoftReference对象清除掉。

Java虚拟机会尽量让软引用存活的时间长一些，迫不得以才清理。
```
import java.lang.ref.SoftReference;
 
public class TestRef {
    public static void main(String args[]) {
        SoftReference<String> str = new SoftReference<String>(new String("abc"));
        System.out.println(str.get());
        //通知JVM进行内存回收
        System.gc();
        System.out.println(str.get());
    }
}
```
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/jvm-2.png)


### 弱引用
在java中，可以用java.lang.ref.WeakReference实例来保存对一个Java对象的弱引用。
```
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
wf.get();//有时候会返回null
wf.isEnQueued();//返回是否被垃圾回收器标记为即将回收的垃圾
```
当GC进行回收时，需要通过算法检查是否回收软引用对象，而对于弱引用对象，GC总是进行回收。弱引用对象更容易，更快的被GC回收。弱引用对象尝尝用于Map结构中，引用数据量比较大的对象，一旦该对象的强引用为null时，GC能够快速的回收该对象空间。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。

```
import java.lang.ref.WeakReference;
 
public class TestRef {
    public static void main(String args[]) {
        WeakReference<String> str = new WeakReference<String>(new String("abc"));
        System.out.println(str.get());
        //通知JVM进行内存回收
        System.gc();
        System.out.println(str.get());
    }
}
```
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/jvm-3.png)


### 虚引用
又称为幽灵引用，主要目的是在一个对象所占的内存被实际回收之前的到通知，从而可以进行一些相关的清理工作。幽灵引用在创建是必须提供一个引用队列作为参数，它的作用在于检测对象是否已经从内存中删除，跟踪垃圾回收过程。其次幽灵引用对象的get方法总是返回null，因此无法通过幽灵引用来获取被引用的对象。
```
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj=null;
pf.get();//永远返回null
pf.isEnQueued();//返回是否从内存中已经删除
```
当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在垃圾回收后，销毁这个对象，将这个虚引用加入引用队列。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

```
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;
 
 
public class TestRef {
    public static void main(String args[]) {
        ReferenceQueue<String> queue = new ReferenceQueue<>();
        PhantomReference<String> str = new PhantomReference<String>("abc", queue);
        System.out.println(str.get());
    }
}
```
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/jvm-4.png)
