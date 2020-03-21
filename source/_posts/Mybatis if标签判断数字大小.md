title: Mybatis if标签判断数字大小
tags:
  - MyBatis
categories:
  - Dev
  - Java
date: 2020-03-18 14:01:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/fish.jpg)
<!-- more -->
## if标签语法
```
<select...>
  SQL语句1
  <if test="条件表达式">
     SQL语句2
  </if>
</select>
```

## 条件表达式中大于号小于号用 gt,lt
```
<if test="num gt 0">...</if>

<if test="num lt 0">...</if>
```

## mapper
```
List<ZftjHalf> selectByAreaIdAndYear(@Param("areaId") String areaId,
                                     @Param("year") String year,
                                     @Param("level") int level);
```

##xml
```
  <select id="selectByAreaIdAndYear" resultType="com.zftdata.nyzft.entity.ZftjHalf">
    select * from ZFTJ_HALF
    where FILLING_TIME LIKE CONCAT(#{year},'%')
    <if test="level lt 3">
      and AREA_ID_PID =#{areaId}
    </if>
    <if test="level gt 2">
      and AREA_ID =#{areaId}
    </if>
    <if test="level == 4">
      and AREA_ID =#{areaId}
    </if>
  </select>
```