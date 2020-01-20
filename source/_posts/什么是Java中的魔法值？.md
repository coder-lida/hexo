title: 什么是Java中的魔法值？
tags:
  - 规范
categories:
  - Java
date: 2020-01-20 08:40:00
cover: true

---
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS04MGU4NTE4NzZmZmY4NTYwLnBuZw?x-oss-process=image/format,png)
<!-- more -->

>使用IDEA时，启用了阿里的代码规范检查，其中就有一项提示是不允许任何魔法值出现在代码里，于是出于好奇就了解一下到底啥时魔法值。

## 介绍
魔法数值、魔法数字、魔法值，这是一个东西，不同的叫法。
所谓魔法值，是指在代码中直接出现的数值，只有在这个数值记述的那部分代码中才能明确了解其含义。

看一段代码
```
   /**
     * 获取当前周所有的日期
     *
     * @return
     */
    public static List<String> getRangeDayOfWeek() {
        List<String> list = new ArrayList<>();
        // 日期格式转换
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        // 时间调整到本周一
        Calendar instance = Calendar.getInstance();
        instance.set(instance.get(Calendar.YEAR), instance.get(Calendar.MONDAY), instance.get(Calendar.DAY_OF_MONTH), 0, 0, 0);
        instance.set(Calendar.DAY_OF_WEEK, Calendar.MONDAY);
        //循环打印
        for (int i = 1; i <= 7; i++) {
            list.add(sdf.format(instance.getTime()));
            instance.add(Calendar.DAY_OF_WEEK, 1);
        }
        return list;
    }
```

## 解决
使用static final 定义常量或使用enum值
```
static final int WEEK_DAYS= 7;
```

>注：使用static final 声明常量，可以方便以后维护更新。修改变量的值时只用修改一处，还不用担心修改了其他不该修改的常量。

## 总结
魔法值的问题对于代码逻辑来说，并不是什么要命的事情，即使不修改也基本不影响代码的正常运行，我以前没有安装阿里代码检查规范时，一样这么使用，也没出现过啥问题。好吧，应该说但是了。但是，遵循公认的代码规范，可以有效的避免开发过程的一些小问题（最让人头疼的往往都是一些小问题引起的），提升开发的效率和代码的可阅读性，老老实实按照规范来，自然就会受益良多，继续加油！