title: Java14新特性：Switch表达式
tags:
  - java14
categories:
  - Dev
  - Java
date: 2020-03-18 14:36:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/img%20%288%29.jpeg)
<!-- more -->
>Java 14正式发布switch表达式特性。在之前的两个 Java 版本Java12，Java13，switch特性只是预览版。
新的switch表达式有助于避免一些bug，因为它的表达和组合方式更容易编写。

switch新的表达式有两个特点：
* 支持箭头表达式返回。
* 支持yied和return返回值。
## Java 14之前switch语法
```
switch (season) {
    case SPRING:
    case AUTUMN:
        System.out.println("温暖");
        break;
    case SUMMER:
        System.out.println("炎热");
        break;
    case WINTER:
        System.out.println("寒冷");
        break;
}
```

## Java 14 switch表达式
```
switch (season) {
    case SPRING, AUTUMN -> System.out.println("温暖");
    case SUMMER         -> System.out.println("炎热");
    case WINTER         -> System.out.println("寒冷");
}
```
Java 14的switch表达式使用箭头表达时，不需要我们在每一个case后都加上break，减少我们出错的机会。

## Java14之前switch语法返回值
```
String temperature ="";
switch (season) {
    case SPRING:
    case AUTUMN:
        temperature  = "温暖";
        break;
    case SUMMER:
        temperature  = "炎热";
        break;
    case WINTER:
        temperature  = "寒冷";
        break;
    default:
       temperature  = "忽冷忽热";
}
```
它不支持返回值，需要通过一个中间变量来返回。

## Java14 switch表达式返回值
```
String temperature = switch (season) {
    case SPRING, AUTUMN -> "温暖";
    case SUMMER         -> "炎热";
    case WINTER         -> "寒冷";
}
```