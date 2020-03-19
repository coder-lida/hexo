title: Java14新特性：增强NPE NullPointerException 错误推断
tags:
  - java14
categories:
  - Dev
  - Java
date: 2020-03-19 08:41:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/java.jpg)
<!-- more -->
>改进 NullPointerExceptions，通过准确描述哪些变量为 null 来提高 JVM 生成的异常的可用性。该提案的作者希望为开发人员和支持人员提供有关程序为何异常终止的有用信息，并通过更清楚地将动态异常与静态程序代码相关联来提高对程序的理解。

```
String name = user.getLocation().getCity().getName();
```
在Java 14之前，你可能会得到如下的错误：
```
Exception in thread "main" java.lang.NullPointerExceptionat NullPointerExample.main(NullPointerExample.java:2)
```
不幸的是，如果在第2行是一个包含了多个方法调用的赋值语句（如getLocation()和getCity()），那么任何一个都可能会返回null。实际上，变量user也可能是null。因此，无法判断是谁导致了NullPointerException。

在Java 14中，新的JVM特性可以显示更详细的诊断信息：
```
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "Location.getCity()" 
because the return value of "User.getLocation()" is null at NullPointerExample.main(NullPointerExample.java:2)
```

该消息包含两个明确的组成部分：
* 后果：`Location.getCity()`无法被调用
* 原因：`User.getLocation()`的返回值为null