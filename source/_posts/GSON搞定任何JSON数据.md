---
title: GSON搞定任何JSON数据
tags:
  - Json
categories:
  - Dev
  - Java
date: 2019-07-26 11:41:00
cover: true

---
![](http://q6pznk9ej.bkt.clouddn.com/bg4.jpeg)
<!-- more -->

## 一、Gson介绍

GSON是Google提供的用来在Java对象和JSON数据之间进行映射的Java类库。可以将一个Json字符转成一个Java对象，或者将一个Java转化为Json字符串。
>*特点:*
* 快速、高效    
* 代码量少、简洁
* 面向对象
* 数据传递和解析

## 二、Gson的pom依赖
```
 <dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.0</version>
 </dependency>
```
## 三、Gson的创建方式

* Gson gson = new gson();

* 通过GsonBuilder()，可以配置多种配置。
```
Gson gson = new GsonBuilder()
                        .setLenient()// json宽松  
                        .enableComplexMapKeySerialization()//支持Map的key为复杂对象的形式  
                        .serializeNulls() //智能null  
                        .setPrettyPrinting()// 调教格式  
                        .disableHtmlEscaping() //默认是GSON把HTML 转义的
                        .create(); 
```

## 四、Gson的基本用法

之前写过一个获取天气参数的API,就需要去解析返回的json数据，就以此为例。
```
 String url = "http://t.weather.sojson.com/api/weather/city/101010100";
 String resultStr = HttpClientUtil.sendGetRequest(url, "UTF-8");
 ```

## 五、进行解析
```
 Gson gson =new Gson();
 Map m= gson.fromJson(resultStr,Map.class);
 System.out.println(m.get("data"));
```
结果
```
{shidu=15%, pm25=15.0, pm10=35.0, quality=优, wendu=3, ganmao=各类人群可自由活动, 
yesterday={date=06, sunrise=07:36, high=高温 3.0℃, low=低温 -7.0℃, sunset=17:03, 
aqi=58.0, ymd=2019-01-06, week=星期日, fx=西南风, fl=<3级, type=晴, 
notice=愿你拥有比阳光明媚的心情}, forecast=[{date=07, sunrise=07:36, high=高温 2.0℃, 
low=低温 -7.0℃, sunset=17:04, aqi=48.0, ymd=2019-01-07, week=星期一, fx=北风, 
fl=3-4级, type=多云, notice=阴晴之间，谨防紫外线侵扰}, {date=08, sunrise=07:36, 
high=高温 1.0℃, low=低温 -9.0℃, sunset=17:05, aqi=28.0, ymd=2019-01-08, week=星期二, 
fx=北风, fl=3-4级, type=晴, notice=愿你拥有比阳光明媚的心情}, {date=09, sunrise=07:36,
 high=高温 2.0℃, low=低温 -8.0℃, sunset=17:06, aqi=83.0, ymd=2019-01-09, week=星期三, 
fx=西南风, fl=<3级, type=多云, notice=阴晴之间，谨防紫外线侵扰}, {date=10, sunrise=07:36, 
high=高温 4.0℃, low=低温 -7.0℃, sunset=17:07, aqi=128.0, ymd=2019-01-10, week=星期四,
 fx=西南风, fl=<3级, type=晴, notice=愿你拥有比阳光明媚的心情}, {date=11, sunrise=07:36, 
high=高温 5.0℃, low=低温 -6.0℃, sunset=17:08, aqi=238.0, ymd=2019-01-11, week=星期五,
 fx=西南风, fl=<3级, type=多云, notice=阴晴之间，谨防紫外线侵扰}]}
可以新建一个天气的Bean，将返回的json数据转换成对象
```
## 六、GSON直接解析成对象
```
ResultBean resultBean = new Gson().fromJson(resultStr,ResultBean.class);
```
## 七、解析简单的json
```
data:{
      shidu = 15 % , 
      pm25 = 15.0,
      pm10 = 35.0, 
      quality = 优, 
      wendu = 3, 
      ganmao = 各类人群可自由活动,
     }
JsonObject jsonObject =(JsonObject) new JsonParser().parse(resultStr);
Int wendu = jsonObject.get("data").getAsJsonObject().get("wendu").getAsInt();
String quality= jsonObject.get("data").getAsJsonObject().get("quality").getAsString();
```
## 八、解析多层对象
```
  data:{
      shidu = 15 % , 
      pm25 = 15.0, 
      pm10 = 35.0, 
      quality = 优, 
      wendu = 3, 
      ganmao = 各类人群可自由活动, 
      yesterday :{
                    date = 06,
                    sunrise = 07: 36,
                    high = 高温 3.0℃,
                    low = 低温 - 7.0℃,
                    sunset = 17: 03,
                    aqi = 58.0,
                    ymd = 2019 - 01 - 06,
                    week = 星期日,
                    fx = 西南风,
                    fl = < 3 级,
                    type = 晴,
                    notice = 愿你拥有比阳光明媚的心情
                 }
        }

 JsonObject jsonObject = (JsonObject) new JsonParser().parse(resultStr);
 JsonObject yesterday = jsonObject.get("data").getAsJsonObject().get("yesterday ").getAsJsonObject();
 String type  = yesterday.get("type").getAsString();
```
## 九、解析带数组的json
```
{
shidu = 15 % , pm25 = 15.0, pm10 = 35.0, quality = 优, wendu = 3, ganmao = 各类人群可自由活动, 
yesterday = {
        date = 06,
        sunrise = 07: 36,
        high = 高温 3.0℃,
        low = 低温 - 7.0℃,
        sunset = 17: 03,
        aqi = 58.0,
        ymd = 2019 - 01 - 06,
        week = 星期日,
        fx = 西南风,
        fl = < 3 级,
        type = 晴,
        notice = 愿你拥有比阳光明媚的心情
    }, 
forecast = [{
        date = 07,
        sunrise = 07: 36,
        high = 高温 2.0℃,
        low = 低温 - 7.0℃,
        sunset = 17: 04,
        aqi = 48.0,
        ymd = 2019 - 01 - 07,
        week = 星期一,
        fx = 北风,
        fl = 3 - 4 级,
        type = 多云,
        notice = 阴晴之间， 谨防紫外线侵扰
    }, {
        date = 08,
        sunrise = 07: 36,
        high = 高温 1.0℃,
        low = 低温 - 9.0℃,
        sunset = 17: 05,
        aqi = 28.0,
        ymd = 2019 - 01 - 08,
        week = 星期二,
        fx = 北风,
        fl = 3 - 4 级,
        type = 晴,
        notice = 愿你拥有比阳光明媚的心情
    }, {
        date = 09,
        sunrise = 07: 36,
        high = 高温 2.0℃,
        low = 低温 - 8.0℃,
        sunset = 17: 06,
        aqi = 83.0,
        ymd = 2019 - 01 - 09,
        week = 星期三,
        fx = 西南风,
        fl = < 3 级,
        type = 多云,
        notice = 阴晴之间， 谨防紫外线侵扰
    }, {
        date = 10,
        sunrise = 07: 36,
        high = 高温 4.0℃,
        low = 低温 - 7.0℃,
        sunset = 17: 07,
        aqi = 128.0,
        ymd = 2019 - 01 - 10,
        week = 星期四,
        fx = 西南风,
        fl = < 3 级,
        type = 晴,
        notice = 愿你拥有比阳光明媚的心情
    }, {
        date = 11,
        sunrise = 07: 36,
        high = 高温 5.0℃,
        low = 低温 - 6.0℃,
        sunset = 17: 08,
        aqi = 238.0,
        ymd = 2019 - 01 - 11,
        week = 星期五,
        fx = 西南风,
        fl = < 3 级,
        type = 多云,
        notice = 阴晴之间， 谨防紫外线侵扰
    }]
}

JsonObject jsonObject =(JsonObject) new JsonParser().parse(resultStr);
//获取data
JsonObject data = jsonObject.get("data").getAsJsonObject();
//获取数组
JsonArray forecast = data.getAsJsonObject().get("forecast").getAsJsonArray();
String type  = forecast.get(0).getAsJsonObject().get("type").getAsString();
```