title: Java中的clone
tags:
  - Java
categories:
  - Java
date: 2020-01-13 07:56:00
cover: true

---
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1hYWRiMDk5MzU4NWNkMjU1LmpwZw?x-oss-process=image/format,png )
<!-- more -->
## Java中对象的创建
* 使用new操作符创建一个对象
* 使用clone方法复制一个对象
### 两种方式的异同
new操作符的本意是分配内存。程序执行到new操作符时， 首先去看new操作符后面的类型，因为知道了类型，才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。
而clone在第一步是和new相似的， 都是分配内存，调用clone方法时，分配的内存和源对象（即调用clone方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，clone方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部 。
## Java中的Clone
clone 顾名思义就是 复制 ， 在Java语言中， clone方法被对象调用，所以会复制对象。所谓的复制对象，首先要分配一个和源对象同样大小的空间，在这个空间中创建一个新的对象
### 复制对象 or 复制引用
```
Person p = new Person(23, "张三");  
Person p1 = p;
System.out.println(p);  
System.out.println(p1); 
```
打印结果：
```
com.pansoft.zhangjg.testclone.Person@2f9ee1ac
com.pansoft.zhangjg.testclone.Person@2f9ee1ac
```
可以看出，打印的地址值是相同的，既然地址都是相同的，那么肯定是同一个对象。p和p1只是引用而已，他们都指向了一个相同的对象Person(23, "张三") 。 可以把这种现象叫做 引用的复制 
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1jOGZkNGRmZWE3ODViODI1LnBuZw?x-oss-process=image/format,png)
而下面的代码是真真正正的克隆了一个对象：
```
Person p = new Person(23, "张三");    
Person p1 = (Person) p.clone();   
System.out.println(p);  
System.out.println(p1);
```
打印结果:
```
com.pansoft.zhangjg.testclone.Person@2f9ee1ac
com.pansoft.zhangjg.testclone.Person@67f1fba0
```
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1mMjkxN2I1NjEzYWFiYTJlLnBuZw?x-oss-process=image/format,png)
## 深拷贝 or 浅拷贝
上面的示例代码中，Person中有两个成员变量，分别是name和age， name是String类型， age是int类型。代码非常简单，如下所示：
```
public class Person implements Cloneable{ 
    private int age ;
    private String name;
    public Person(int age, String name) {
       this.age = age; 
       this.name = name;  
    }
   public Person() {}  
   public int getAge() {
       return age;
   }
  public String getName() {
       return name;
   } 
 @Override
 protected Object clone() throws CloneNotSupportedException{
    return (Person)super.clone();
 }
}
```
由于age是基本数据类型， 那么对它的拷贝没有什么疑议，直接将一个4字节的整数值拷贝过来就行。但是name是String类型的， 它只是一个引用， 指向一个真正的String对象，那么对它的拷贝有两种方式：

①直接将源对象中的name的引用值拷贝给新对象的name字段；

②根据原Person对象中的name指向的字符串对象创建一个新的相同的字符串对象，将这个新字符串对象的引用赋给新拷贝的Person对象的name字段。

这两种拷贝方式分别叫做 浅拷贝 和 深拷贝 。

深拷贝和浅拷贝的原理如下图所示：
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0wZWNkYTQ4MDAwNzMyM2U5LnBuZw?x-oss-process=image/format,png)
## clone是浅拷贝还是深拷贝
如果两个Person对象的name的地址值相同， 说明两个对象的name都指向同一个String对象， 也就是浅拷贝， 而如果两个对象的name的地址值不同， 那么就说明指向不同的String对象， 也就是在拷贝Person对象的时候， 同时拷贝了name引用的String对象， 也就是深拷贝。验证代码如下：
```
Person p = new Person(23,"张三");
Person p1 =(Person)p.clone();
String  result = p.getName() == p1.getName() ? 
                 "clone是浅拷贝的":"clone是深拷贝的";
```
打印结果:
```
clone是浅拷贝的
```

