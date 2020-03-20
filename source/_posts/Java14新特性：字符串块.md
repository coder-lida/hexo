title: Java14新特性：字符串块
tags:
  - java14
categories:
  - Dev
  - Java
date: 2020-03-20 14:04:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/java.jpg)
<!-- more -->
java1之前写字符串拼接
```
 String str = "<html>" +
                "<header>" +
                "</header>" +
                "<body>" +
                "<div>body</div>" +
                "</body>" +
                "</html>";
```
内容短的时候还算可以，当需要拼接的内容很多的时候就会显得很乱

java14后，引进了三个引号作为字符串块，类似python中的字符串块
```
        // 注意 """ 之后必须换行
        String str = """
                <html>
                        <header>
                        </header>
                    <body>
                        <div>"body"</div>
                    </body>
                </html>
                """;
```