title: Java14新特性：增强 instanceOf 类型推断
tags:
  - java14
categories:
  - Dev
  - Java
date: 2020-03-14 08:17:00
cover: true

---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/java.png)
<!-- more -->
>Java中instanceof是用来判断对象的类型是否是目标类型。如果是返回true，不是返回false。

在Java 14之前，示例如下：
```
if (obj instanceof String) {
    String str = (String) obj; 
    str.contains("A") ;
}else{
     str = "";
}
```
obj instanceof String已经为true，在后面的代码里，我们还是要清晰的定义一个新变量，并且要做类型强转换。

Java 14对instanceof引入了模式匹配，修改后的代码如下：
```
if (!(obj instanceof String str)) {
     str.contains("A") ;
} else {
     str = "";
}
```
定义了str，就可以在后续代码使用，不在需要显式做类型转换了。

还能继续加入判断条件
```
if (obj instanceof String str && str.contains("A")) {
            System.out.println(str);
        }

if (obj instanceof String str || str.contains("A")) {
            System.out.println(str);
        }
```