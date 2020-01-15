title: 记一次jsoup的使用
tags:
  - 爬虫
categories:
  - Java
date: 2019-07-06 15:46:00
cover: true

---
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xMjFjNDJkZGU1OGRjZTdlLnBuZw?x-oss-process=image/format,png )
<!-- more -->

Jsoup是用于解析HTML，就类似XML解析器用于解析XML。 Jsoup它解析HTML成为真实世界的HTML。 它与jquery选择器的语法非常相似，并且非常灵活容易使用以获得所需的结果。

## 安装
### 依赖
```
<dependency>
  <!-- jsoup HTML parser library @ http://jsoup.org/ -->
  <groupId>org.jsoup</groupId>
  <artifactId>jsoup</artifactId>
  <version>1.10.2</version>
</dependency>
```
## 应用
### 从URL获取HTML来解析
```
Document doc = Jsoup.connect("http://www.baidu.com/").get();
String title = doc.title();
```
connect(String url) 方法创建一个新的 Connection, 和 get() 取得和解析一个HTML文件。如果从该URL获取HTML时发生错误，便会抛出 IOException，应适当处理。
Connection 接口还提供一个方法链来解决特殊请求，具体如下
```
  Document doc = Jsoup.connect("http://example.com";)
  .data("query", "Java")
  .userAgent("Mozilla")
  .cookie("auth", "token")
  .timeout(3000)
  .post();
```
### 查看元素
```
getElementById(String id);
getElementsByTag(String tag);
getElementsByClass(String className);
getElementsByAttribute(String key) (and related methods);
Element siblings: siblingElements(), firstElementSibling(), lastElementSibling(); nextElementSibling(), previousElementSibling();
Graph: parent(), children(), child(int index);
```
### 元素数据
```
attr(String key)获取属性attr(String key, String value)设置属性
attributes()获取所有属性
id(), className() and classNames()
text()获取文本内容text(String value) 设置文本内容
html()获取元素内HTMLhtml(String value)设置元素内的HTML内容
outerHtml()获取元素外HTML内容
data()获取数据内容（例如：script和style标签)
tag() and tagName()
```
### 操作HTML和文本
```
append(String html), prepend(String html)
appendText(String text), prependText(String text)
appendElement(String tagName), prependElement(String tagName)
html(String value)
```
### 通过类似于css或jQuery的选择器来查找元素
```
Elements trs = doc.select(".kuang").select("tbody").get(5).select("tr");
        StringBuilder controlTarget = new StringBuilder();
        for (int i = 0; i < trs.size(); i++) {
            if (i >= 1 && i < trs.size() - 1) {
                Elements tds = trs.get(i).select("td");
                res.setCropRange(tds.get(0).text());
                res.setDosage(tds.get(2).text());
                res.setMethod(tds.get(3).text());
                controlTarget.append(tds.get(1).text()).append(" ");
            }
        }
```
### Selector选择器概述

```
   tagname: 通过标签查找元素，比如：a;
   ns|tag: 通过标签在命名空间查找元素，比如：可以用 fb|name 语法来查找 <fb:name> 元素;
   '#id': 通过ID查找元素，比如：#logo;
   .class: 通过class名称查找元素，比如：.masthead;
   [attribute]: 利用属性查找元素，比如：[href];
   [^attr]: 利用属性名前缀来查找元素，比如：可以用[^data-] 来查找带有HTML5 Dataset属性的元素;
   [attr=value]: 利用属性值来查找元素，比如：[width=500];
   [attr^=value], [attr$=value], [attr*=value]: 利用匹配属性值开头、结尾或包含属性值来查找元素，比如：[href*=/path/];
   [attr~=regex]: 利用属性值匹配正则表达式来查找元素，比如： img[src~=(?i)\.(png|jpe?g)];
   *: 这个符号将匹配所有元素;
```
### Selector选择器组合使用
```
   el#id: 元素+ID，比如： div#logo;
   el.class: 元素+class，比如： div.masthead;
   el[attr]: 元素+class，比如： a[href];
   任意组合，比如：a[href].highlight;
   ancestor child: 查找某个元素下子元素，比如：可以用.body p 查找在"body"元素下的所有 p元素;
   parent > child: 查找某个父元素下的直接子元素，比如：可以用div.content > p 查找 p 元素，也可以用body > * 查找body标签下所有直接子元素;
   siblingA + siblingB: 查找在A元素之前第一个同级元素B，比如：div.head + div;
   siblingA ~ siblingX: 查找A元素之前的同级X元素，比如：h1 ~ p;
   el, el, el:多个选择器组合，查找匹配任一选择器的唯一元素，例如：div.masthead, div.logo;
```
### 伪选择器selectors
```
:lt(n): 查找哪些元素的同级索引值（它的位置在DOM树中是相对于它的父节点）小于n，比如：td:lt(3) 表示小   于三列的元素
   :gt(n):查找哪些元素的同级索引值大于n，比如： div p:gt(2)表示哪些div中有包含2个以上的p元素
   :eq(n): 查找哪些元素的同级索引值与n相等，比如：form input:eq(1)表示包含一个input标签的Form元素
   :has(seletor): 查找匹配选择器包含元素的元素，比如：div:has(p)表示哪些div包含了p元素
   :not(selector): 查找与选择器不匹配的元素，比如： div:not(.logo) 表示不包含 class="logo" 元素的所有 div 列表
   :contains(text): 查找包含给定文本的元素，搜索不区分大不写，比如： p:contains(jsoup)
   :containsOwn(text): 查找直接包含给定文本的元素
   :matches(regex): 查找哪些元素的文本匹配指定的正则表达式，比如：div:matches((?i)login)
   :matchesOwn(regex): 查找自身包含文本匹配指定正则表达式的元素
```
### 提取给定URL中的链接
```
            Document doc = Jsoup.connect("http://www.yiibai.com").get();  
            Elements links = doc.select("a[href]");  
            for (Element link : links) {  
                System.out.println("\nlink : " + link.attr("href"));  
                System.out.println("text : " + link.text());  
            }  
```
### 提取URL中的元数据

```
            Document doc = Jsoup.connect("http://www.yiibai.com").get();  
            String keywords = doc.select("meta[name=keywords]").first().attr("content");  
            System.out.println("Meta keyword : " + keywords);  
            String description = doc.select("meta[name=description]").get(0).attr("content");  
            System.out.println("Meta description : " + description);  

```
### 提取URL中的图像
```
            Document doc = Jsoup.connect("http://www.yiibai.com").get();  
            Elements images = doc.select("img[src~=(?i)\\.(png|jpe?g|gif)]");  
            for (Element image : images) {  
                System.out.println("src : " + image.attr("src"));  
                System.out.println("height : " + image.attr("height"));  
                System.out.println("width : " + image.attr("width"));  
                System.out.println("alt : " + image.attr("alt"));  
            }  
```