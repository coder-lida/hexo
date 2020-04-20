title: Java并发编程(一)CAS
tags:
  - CAS
  - 并发
categories:
  - Dev
  - Java
date: 2020-04-20 08:06:00
cover: true
---
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/Jennifer-2.png)
<!-- more -->
## CAS 是什么
CAS 的全称 Compare-And-Swap，它是一条 CPU 并发。

它的功能是判断内存某一个位置的值是否为预期，如果是则更改这个值，这个过程就是原子的。

CAS 并发原体现在 JAVA 语言中就是 `sun.misc.Unsafe` 类中的各个方法。调用 `UnSafe` 类中的 CAS 方法，JVM 会帮我们实现出 CAS 汇编指令。这是一种完全依赖硬件的功能，通过它实现了原子操作。由于 CAS 是一种系统源语，源语属于操作系统用语范畴，是由若干条指令组成，用于完成某一个功能的过程，并且原语的执行必须是连续的，在执行的过程中不允许被中断，也就是说 CAS 是一条原子指令，不会造成所谓的数据不一致的问题。 

## 比较并交换
CAS的意思就是比较并交换。上面说到，这个比较过程是原子的。我们新建一个测试类。
```
public class CASDemo {
    public static void main(String[] args) {
       checkCAS();
    }

    public static void checkCAS(){
        AtomicInteger atomicInteger = new AtomicInteger(5);
        System.out.println(atomicInteger.compareAndSet(5, 2019) + "\t current data is 
        " + atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 2020) + "\t current data is 
        " + atomicInteger.get());
    }
    atomicInteger.getAndIncrement();
    System.out.println("current data is " + atomicInteger.get());
}
```
查看返回结果
```
true	 current data is 2019
false	 current data is 2019
current data is 2020
```
原子整型类的初始值是5，当第一次调用compareAndSet的时候期望值是5，更新值是2019，此时的期望值和atomicInteger 值相等，则替换为更新值，输出为2019；第二次调用compareAndSet的时候期望值还是5，此时atomicInteger的值已经更新为2019，期望值和原始值不想等，不做更新操作，所以此时的atomicInteger值还是2019。

compareAndSet是AtomicInteger的一个方法
```
/**
     * Atomically sets the value to the given updated value
     * if the current value {@code ==} the expected value.
     *
     * @param expect the expected value
     * @param update the new value
     * @return {@code true} if successful. False return indicates that
     * the actual value was not equal to the expected value.
     */
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```
他调用的是unsafe类的`compareAndSwapInt`方法，this表示当前值对象，valueOffset是当前对象在内存中的偏移量，expect为期望值，update为更新值。

## 原子性
需要说到 `atomicInteger.getAndIncrement();`这个方法，类似于i++。
```
    /**
     * Atomically increments by one the current value.
     *
     * @return the previous value
     */
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
```
也是调用的unsafe类的方法。
来看一下`getAndAddInt`：
```
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```
var1为当前对象，var2为当前对象在内存中的偏移量，var4为1，var5为`getIntVolatile(var1, var2)`的返回值，`getIntVolatile`方法的意思是当前对象var1且内存偏移量为var2时的值是多少。

在while循环中，同样调用了`compareAndSwapInt`方法，此时的var5为期望值，var5+var4为更新值。直到比较成功。

## Unsafe类

unsafe类是CAS的核心类，由于java无法直接访问底层系统，需要通过本地（native）方法来访问，基于unsafe类可直接操作特定内存的数据unsafe类存在于sun.mics包中，其内部方法可以像c的指针一样直接操作内存。因为 Java 中 CAS 操作执行依赖于 Unsafe 类。 

变量 vauleOffset，表示该变量值在内存中的偏移量，因为 Unsafe 就是根据内存偏移量来获取数据的。

变量 value 用 volatile 修饰，保证了多线程之间的内存可见性。
```
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            // 获取下面 value 的地址偏移量
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
	// ...
}
```

## CAS 的缺点


* 循环时间长开销很大
  * 如果 CAS 失败，会一直尝试，如果 CAS 长时间一直不成功，可能会给 CPU 带来很大的开销（比如线程数很多，每次比较都是失败，就会一直循环），所以希望是线程数比较小的场景。
* 只能保证一个共享变量的原子操作
  * 对于多个共享变量操作时，循环 CAS 就无法保证操作的原子性。
* 引出 ABA 问题

## ABA 问题

### 原子引用
```
public class AtomicRefrenceDemo {
    public static void main(String[] args) {
        User z3 = new User("张三", 22);
        User l4 = new User("李四", 23);
        AtomicReference<User> atomicReference = new AtomicReference<>();
        atomicReference.set(z3);
        System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());
        System.out.println(atomicReference.compareAndSet(z3, l4) + "\t" + atomicReference.get().toString());
    }
}

@Getter
@ToString
@AllArgsConstructor
class User {
    String userName;
    int age;
}
```

### ABA 问题是怎么产生的
```
public class ABADemo {
    private static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);

    public static void main(String[] args) {
        new Thread(() -> {
            atomicReference.compareAndSet(100, 101);
            atomicReference.compareAndSet(101, 100);
        }).start();

        new Thread(() -> {
            // 保证上面线程先执行
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicReference.compareAndSet(100, 2019);
            System.out.println(atomicReference.get()); // 2019
        }).start();
    }
}
```
当有一个值从 A 改为 B 又改为 A，这就是 ABA 问题。

### ABA 问题解决
时间戳原子引用
```
public class ABADemo2 {
    private static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + " 的版本号为：" + stamp);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicStampedReference.compareAndSet(100, 101, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1 );
            atomicStampedReference.compareAndSet(101, 100, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1 );
        }).start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + " 的版本号为：" + stamp);
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean b = atomicStampedReference.compareAndSet(100, 2019, stamp, stamp + 1);
            System.out.println(b); // false
            System.out.println(atomicStampedReference.getReference()); // 100
        }).start();
    }
}
```
输出结果
```
Thread-0 的版本号为：1
Thread-1 的版本号为：1
false
100
```

