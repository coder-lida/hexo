title: Java反射
tags:
  - 反射
categories:
  - 技术
date: 2020-01-06 07:57:00
cover: true

---

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1hNGQzN2EyYjc0MWZiOGVhLmpwZw?x-oss-process=image/format,png)
<!-- more -->
## 一、什么是反射？
[JAVA反射机制](https://baike.baidu.com/item/JAVA%E5%8F%8D%E5%B0%84%E6%9C%BA%E5%88%B6/6015990)是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。
>简单的来说：
1.通过new关键字创建对象操作对象，在编译时就已经确定。
2.通过反射可以在程序运行过程中动态的操作对象，可以获得编译期无法获得的信息，动态操作最大限度发挥了java扩展性。

## 二、反射原理
Java反射的原理:java类的执行需要经历以下过程：
* 编译：.java文件编译后生成.class字节码文件
* 加载：类加载器负责根据一个类的全限定名来读取此类的二进制字节流到JVM内部，并存储在运行时内存区的方法区，然后将其转换为一个与目标类型对应的java.lang.Class对象实例
* 链接
 `验证`：格式（class文件规范） 语义（final类是否有子类） 操作
`准备`：静态变量赋初值和内存空间，final修饰的内存空间直接赋原值，此处不是用户指定的初值。
`解析`：符号引用转化为直接引用，分配地址
* 初始化：有父类先初始化父类，然后初始化自己；将static修饰代码执行一遍，如果是静态变量，则用用户指定值覆盖原有初值；如果是代码块，则执行一遍操作。

Java的反射就是利用上面第二步加载到jvm中的.class文件来进行操作的。.class文件中包含java类的所有信息，当你不知道某个类具体信息时，可以使用反射获取class，然后进行各种操作。

Java反射就是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；并且能改变它的属性。总结说：反射就是把java类中的各种成分映射成一个个的Java对象，并且可以进行操作。

## 三、反射机制相关
与Java反射相关的类如下：

| 类名| 用途|
|-----|-----|
| Class类 | 代表类的实体，在运行的Java应用程序中表示类和接口|
| Field类 | 代表类的成员变量（成员变量也称为类的属性）|
| Method类| 代表类的方法|
| Constructor类 | 代表类的构造方法|

 反射可访问的常用信息
 
| 类型| 访问方法|返回值类型|说明|
|-----|-----|-----|-----|
| 包路径 | getPackage()|Package 对象| 获取该类的存放路径|
| 类名称| getName()|String 对象| 获取该类的名称|
| 继承类| getSuperclass()|Class 对象|  获取该类继承的类|
| 实现接口| getlnterfaces()|Class 型数组|  获取该类实现的所有接口|
| 构造方法| getConstructors()|Constructor 型数组|  获取所有权限为 public 的构造方法|
| 构造方法| getDeclaredContruectors()|Constructor 对象|  获取当前对象的所有构造方法|
| 方法| getMethods()|Methods 型数组|  获取所有权限为 public 的方法|
| 方法| getDeclaredMethods()|Methods 对象|  获取当前对象的所有方法|
| 成员变量| getFields()|Field 型数组|  获取所有权限为 public 的成员变量|
| 成员变量| getDeclareFileds()|Field 对象|  获取当前对象的所有成员变量|
| 内部类| getClasses()|Class 型数组|  获取所有权限为 public 的内部类|
| 内部类| getDeclaredClasses()|Class 型数组|  获取所有内部类|
| 内部类的声明类| getDeclaringClass()|Class 对象|  如果该类为内部类，则返回它的成员类，否则返回 null|

Java 反射机制主要提供了以下功能，这些功能都位于java.lang.reflect包。

* 在运行时判断任意一个对象所属的类。

* 在运行时构造任意一个类的对象。

* 在运行时判断任意一个类所具有的成员变量和方法。

* 在运行时调用任意一个对象的方法。

* 生成动态代理。

## 四、反射的使用
### 1、java中的Class三种获取方式

　jdk提供了三种方式获取一个对象的Class，就Person person 来说

　　1.person .getClass()，这个是Object类里面的方法

　　2.Person .Class属性，任何的数据类型，基本数据类型或者抽象数据类型，都可以通过这种方式获取类

　　3.Class.forName("")，Class类提供了这样一个方法，让我们通过类名来获取到对象类

　说明：在运行期间，如果我们要产生某个类的对象，Java虚拟机(JVM)会检查该类型的Class对象是否已被加载。如果没有被加载，JVM会根据类的名称找到.class文件并加载它。一旦某个类型的Class对象已被加载到内存，就可以用它来产生该类型的所有对象。 
```
    //方式一
    Person person = new Person();
    Class<? extends Person> personClazz01 = person.getClass();
 
    //方式二
    try {
        Class<?> personClazz02 = Class.forName("Person");
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
 
    //方式三
    Class<? extends Person> personClazz03 = Person.class;
 
```
### 2、如何通过反射获取私有成员变量和私有方法
Person类 
```
public class Person {
private String name = "zhangsan";
private String age;
 
public String getName() {
    return name;
}
 
public void setName(String name) {
    this.name = name;
}
}  
 
 
    Person person = new Person();
    //打印没有改变属性之前的name值
    System.out.println("before：" + getPrivateValue(person, "name"));
    person.setName("lisi");
    //打印修改之后的name值
    System.out.println("after：" + getPrivateValue(person, "name"));
 
 
 
/**
 * 通过反射获取私有的成员变量
 *
 * @param person
 * @return
 */
private Object getPrivateValue(Person person, String fieldName) {
 
    try {
        Field field = person.getClass().getDeclaredField(fieldName);
        // 参数值为true，打开禁用访问控制检查
        //setAccessible(true) 并不是将方法的访问权限改成了public，而是取消java的权限控制检查。
        //所以即使是public方法，其accessible 属相默认也是false
        field.setAccessible(true);
        return field.get(person);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
} 
```
运行结果
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1iNmRjM2Q1MWY1YTM5NzA1LnBuZw?x-oss-process=image/format,png)
### 3、demo
```
package cn.lee.demo;
 
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.lang.reflect.TypeVariable;
 
public class Main {
	/**
	 * 为了看清楚Java反射部分代码，所有异常我都最后抛出来给虚拟机处理！
	 * @param args
	 * @throws ClassNotFoundException
	 * @throws InstantiationException
	 * @throws IllegalAccessException
	 * @throws InvocationTargetException 
	 * @throws IllegalArgumentException 
	 * @throws NoSuchFieldException 
	 * @throws SecurityException 
	 * @throws NoSuchMethodException 
	 */
	public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException, SecurityException, NoSuchFieldException, NoSuchMethodException {
		// TODO Auto-generated method stub
		
		//Demo1.  通过Java反射机制得到类的包名和类名
		Demo1();
		System.out.println("===============================================");
		
		//Demo2.  验证所有的类都是Class类的实例对象
		Demo2();
		System.out.println("===============================================");
		
		//Demo3.  通过Java反射机制，用Class 创建类对象[这也就是反射存在的意义所在]，无参构造
		Demo3();
		System.out.println("===============================================");
		
		//Demo4:  通过Java反射机制得到一个类的构造函数，并实现构造带参实例对象
		Demo4();
		System.out.println("===============================================");
		
		//Demo5:  通过Java反射机制操作成员变量, set 和 get
		Demo5();
		System.out.println("===============================================");
		
		//Demo6: 通过Java反射机制得到类的一些属性： 继承的接口，父类，函数信息，成员信息，类型等
		Demo6();
		System.out.println("===============================================");
		
		//Demo7: 通过Java反射机制调用类中方法
		Demo7();
		System.out.println("===============================================");
		
		//Demo8: 通过Java反射机制获得类加载器
		Demo8();
		System.out.println("===============================================");
		
	}
	
	/**
	 * Demo1: 通过Java反射机制得到类的包名和类名
	 */
	public static void Demo1()
	{
		Person person = new Person();
		System.out.println("Demo1: 包名: " + person.getClass().getPackage().getName() + "，" 
				+ "完整类名: " + person.getClass().getName());
	}
	
	/**
	 * Demo2: 验证所有的类都是Class类的实例对象
	 * @throws ClassNotFoundException 
	 */
	public static void Demo2() throws ClassNotFoundException
	{
		//定义两个类型都未知的Class , 设置初值为null, 看看如何给它们赋值成Person类
		Class<?> class1 = null;
        Class<?> class2 = null;
        
        //写法1, 可能抛出 ClassNotFoundException [多用这个写法]
        class1 = Class.forName("cn.lee.demo.Person");
        System.out.println("Demo2:(写法1) 包名: " + class1.getPackage().getName() + "，" 
				+ "完整类名: " + class1.getName());
        
        //写法2
        class2 = Person.class;
        System.out.println("Demo2:(写法2) 包名: " + class2.getPackage().getName() + "，" 
				+ "完整类名: " + class2.getName());
	}
	
	/**
	 * Demo3: 通过Java反射机制，用Class 创建类对象[这也就是反射存在的意义所在]
	 * @throws ClassNotFoundException 
	 * @throws IllegalAccessException 
	 * @throws InstantiationException 
	 */
	public static void Demo3() throws ClassNotFoundException, InstantiationException, IllegalAccessException
	{
		Class<?> class1 = null;
		class1 = Class.forName("cn.lee.demo.Person");
		//由于这里不能带参数，所以你要实例化的这个类Person，一定要有无参构造函数哈～
		Person person = (Person) class1.newInstance();
		person.setAge(20);
		person.setName("LeeFeng");
		System.out.println("Demo3: " + person.getName() + " : " + person.getAge());
	}
	
	/**
	 * Demo4: 通过Java反射机制得到一个类的构造函数，并实现创建带参实例对象
	 * @throws ClassNotFoundException 
	 * @throws InvocationTargetException 
	 * @throws IllegalAccessException 
	 * @throws InstantiationException 
	 * @throws IllegalArgumentException 
	 */
	public static void Demo4() throws ClassNotFoundException, IllegalArgumentException, InstantiationException, IllegalAccessException, InvocationTargetException
	{
		Class<?> class1 = null;
		Person person1 = null;
		Person person2 = null;
		
		class1 = Class.forName("cn.lee.demo.Person");
		//得到一系列构造函数集合
		Constructor<?>[] constructors = class1.getConstructors();
		
		person1 = (Person) constructors[0].newInstance();
		person1.setAge(30);
		person1.setName("leeFeng");
		
		person2 = (Person) constructors[1].newInstance(20,"leeFeng");
		
		System.out.println("Demo4: " + person1.getName() + " : " + person1.getAge()
				+ "  ,   " + person2.getName() + " : " + person2.getAge()
				);
		
	}
	
	/**
	 * Demo5: 通过Java反射机制操作成员变量, set 和 get
	 * 
	 * @throws IllegalAccessException 
	 * @throws IllegalArgumentException 
	 * @throws NoSuchFieldException 
	 * @throws SecurityException 
	 * @throws InstantiationException 
	 * @throws ClassNotFoundException 
	 */
	public static void Demo5() throws IllegalArgumentException, IllegalAccessException, SecurityException, NoSuchFieldException, InstantiationException, ClassNotFoundException
	{
		Class<?> class1 = null;
		class1 = Class.forName("cn.lee.demo.Person");
		Object obj = class1.newInstance();
		
		Field personNameField = class1.getDeclaredField("name");
		personNameField.setAccessible(true);
		personNameField.set(obj, "胖虎先森");
		
		
		System.out.println("Demo5: 修改属性之后得到属性变量的值：" + personNameField.get(obj));
		
	}
	
 
	/**
	 * Demo6: 通过Java反射机制得到类的一些属性： 继承的接口，父类，函数信息，成员信息，类型等
	 * @throws ClassNotFoundException 
	 */
	public static void Demo6() throws ClassNotFoundException
	{
		Class<?> class1 = null;
		class1 = Class.forName("cn.lee.demo.SuperMan");
		
		//取得父类名称
		Class<?>  superClass = class1.getSuperclass();
		System.out.println("Demo6:  SuperMan类的父类名: " + superClass.getName());
		
		System.out.println("===============================================");
		
		
		Field[] fields = class1.getDeclaredFields();
		for (int i = 0; i < fields.length; i++) {
			System.out.println("类中的成员: " + fields[i]);
		}
		System.out.println("===============================================");
		
		
		//取得类方法
		Method[] methods = class1.getDeclaredMethods();
		for (int i = 0; i < methods.length; i++) {
			System.out.println("Demo6,取得SuperMan类的方法：");
			System.out.println("函数名：" + methods[i].getName());
			System.out.println("函数返回类型：" + methods[i].getReturnType());
			System.out.println("函数访问修饰符：" + Modifier.toString(methods[i].getModifiers()));
			System.out.println("函数代码写法： " + methods[i]);
		}
		
		System.out.println("===============================================");
		
		//取得类实现的接口,因为接口类也属于Class,所以得到接口中的方法也是一样的方法得到哈
		Class<?> interfaces[] = class1.getInterfaces();
		for (int i = 0; i < interfaces.length; i++) {
			System.out.println("实现的接口类名: " + interfaces[i].getName() );
		}
		
	}
	
	/**
	 * Demo7: 通过Java反射机制调用类方法
	 * @throws ClassNotFoundException 
	 * @throws NoSuchMethodException 
	 * @throws SecurityException 
	 * @throws InvocationTargetException 
	 * @throws IllegalAccessException 
	 * @throws IllegalArgumentException 
	 * @throws InstantiationException 
	 */
	public static void Demo7() throws ClassNotFoundException, SecurityException, NoSuchMethodException, IllegalArgumentException, IllegalAccessException, InvocationTargetException, InstantiationException
	{
		Class<?> class1 = null;
		class1 = Class.forName("cn.lee.demo.SuperMan");
		
		System.out.println("Demo7: \n调用无参方法fly()：");
		Method method = class1.getMethod("fly");
		method.invoke(class1.newInstance());
		
		System.out.println("调用有参方法walk(int m)：");
		method = class1.getMethod("walk",int.class);
		method.invoke(class1.newInstance(),100);
	}
	
	/**
	 * Demo8: 通过Java反射机制得到类加载器信息
	 * 
	 * 在java中有三种类类加载器。[这段资料网上截取]
		1）Bootstrap ClassLoader 此加载器采用c++编写，一般开发中很少见。
		2）Extension ClassLoader 用来进行扩展类的加载，一般对应的是jre\lib\ext目录中的类
		3）AppClassLoader 加载classpath指定的类，是最常用的加载器。同时也是java中默认的加载器。
	 * 
	 * @throws ClassNotFoundException 
	 */
	public static void Demo8() throws ClassNotFoundException
	{
		Class<?> class1 = null;
		class1 = Class.forName("cn.lee.demo.SuperMan");
		String nameString = class1.getClassLoader().getClass().getName();
		
		System.out.println("Demo8: 类加载器类名: " + nameString);
	}
	
	
	
}
/**
 * 
 * @author xiaoyaomeng
 *
 */
class  Person{
	private int age;
	private String name;
	public Person(){
		
	}
	public Person(int age, String name){
		this.age = age;
		this.name = name;
	}
 
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}
 
class SuperMan extends Person implements ActionInterface
{
	private boolean BlueBriefs;
	
	public void fly()
	{
		System.out.println("超人会飞耶～～");
	}
	
	public boolean isBlueBriefs() {
		return BlueBriefs;
	}
	public void setBlueBriefs(boolean blueBriefs) {
		BlueBriefs = blueBriefs;
	}
 
	@Override
	public void walk(int m) {
		// TODO Auto-generated method stub
		System.out.println("超人会走耶～～走了" + m + "米就走不动了！");
	}
}
interface ActionInterface{
	public void walk(int m);
}
```
## 五、java反射调用service或mapper中的接口
java中的反射需要一个实例，但是接口无法提供这样的实例，但是JDK提供了一个叫做动态代理的东西，这个代理恰恰只能代理接口。所以我们想要反射接口需要使用这个动态代理来做。

在java的动态代理机制中，有两个重要的东西，一个是 InvocationHandler(接口)、另一个则是 Proxy(类)，这是我们动态代理必须用到的两个东西。

### 1、静态代理
先来看一下静态代理
```
public class TestStaticProxy {
    //这里传入的是接口类型的对象，方便向上转型，实现多态
    public static void consumer(ProxyInterface pi){
        pi.say();
    }
    public static void main(String[] args) {
        // TODO Auto-generated method stub
        consumer(new ProxyObject());
    }
}

//代理接口
interface ProxyInterface{
    public void say();
}


//被代理者
class RealObject implements ProxyInterface{
    //实现接口方法
    @Override
    public void say() {
        // TODO Auto-generated method stub
        System.out.println("say");
    }
    
}


//代理者
class ProxyObject implements ProxyInterface{

    @Override
    public void say() {
        // TODO Auto-generated method stub
        //dosomething for example
        System.out.println("hello proxy");
        new RealObject().say();
        System.out.println("this is method end");
    }
    
}
output:
hello proxy
say
this is method end
```
### 2、动态代理
```
import java.lang.reflect.*;

public class TestActiveProxy{
    static void customer(ProxyInterface pi){
        pi.say();
    }
    public static void main(String[] args){
        RealObject real = new RealObject();
        ProxyInterface proxy = (ProxyInterface)Proxy.newProxyInstance(ProxyInterface.class.getClassLoader(),new Class[]{ProxyInterface.class}, new ProxyObject(real));
        customer(proxy);
    }
}


interface ProxyInterface{
    void say();
}

//被代理类
class RealObject implements ProxyInterface{
    public void say(){
        System.out.println("i'm talking");
    }
}

//代理类，实现InvocationHandler 接口
class ProxyObject implements InvocationHandler{
    private Object proxied = null;
    public ProxyObject(){
        
    }
    public ProxyObject(Object proxied){
        this.proxied  = proxied;
    }
    public Object invoke(Object arg0, Method arg1, Object[] arg2) throws Throwable {
        System.out.println("hello");
        return arg1.invoke(proxied, arg2);
    };
}
```

### 3、应用场景
假如现在我们需要通过反射得到`TestMapper`接口，然后调用其中的一个`selectById`方法
```
    public interface TestMapper{
            /**
            * 根据id查对象
            */
            User  selectById(@Param("id") Integer id);
    }
```
现在如果我们需要反射使用该接口根据用户ID获取用户对象是无法直接反射调取的，所以我们需要一个动态代理类。
创建一个`MyInvocationHandler`实现`InvocationHandler`接口
```
public class MyInvocationHandler implements InvocationHandler {

    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(target,args);
    }
}
```
去生成代理对象并调用方法
```
 SqlSession sqlSession = this.sqlSessionFactory.openSession();

 Class<?> clazz = Class.forName("com.example.demo.mapper.TestMapper");

 Object instance = Proxy.newProxyInstance(
                clazz.getClassLoader(),
                new Class[]{clazz},
                new MyInvocationHandler(sqlSession.getMapper(clazz))
        );
//这里我是通过sqlSession来获取Mapper的

 Method method = instance.getClass().getMethod("selectById",Integer.class);
 method.invoke(instance, 1);
//object为mapper中传入的参数
```

这里需要注意，newProxyInstance()方法中最后一个参数，即为我们创建的动态代理的类（因为我这里调用的接口为mybatis中mapper中的接口，所以需要从sqlSession中getMapper）。


参考：
http://blog.qiji.tech/archives/4374
https://www.jianshu.com/p/9be58ee20dee
https://blog.csdn.net/ljphhj/article/details/12858767
https://developer.android.google.cn/reference/java/lang/reflect/Method?hl=zh-cn

