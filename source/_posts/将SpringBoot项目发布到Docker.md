title: 将SpringBoot项目发布到Docker
tags:
  - 部署
categories:
  - 技术
date: 2020-03-10 19:33:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/fish.jpg)
<!-- more -->
### 1.创建springboot项目
![](http://q6rnahf7l.bkt.clouddn.com/springboot.png)

```
package com.test.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @RequestMapping("/")
    @ResponseBody
    public String hello() {
        return "Hello, SpringBoot With Docker";
    }
}

```

### 2.将SpringBoot项目打jar包
`pom.xml`增加`spring-boot-maven-plugin`插件

使用右侧`maven-Lifecycle-package`打jar包

使用`java -jar *-1.0.0.jar`测试可用


### 3.编写Dockerfile文件
```
# 基础镜像使用java
FROM java:8
# VOLUME 指定了临时文件目录为/tmp。
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp 
# 将jar包添加到容器中并更名为app.jar
ADD demo-0.0.1-SNAPSHOT.jar app.jar 
# 运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]

```

解释下这个配置文件：

VOLUME 指定了临时文件目录为/tmp。其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp。改步骤是可选的，如果涉及到文件系统的应用就很有必要了。/tmp目录用来持久化到 Docker 数据文件夹，因为 Spring Boot 使用的内嵌 Tomcat 容器默认使用/tmp作为工作目录
项目的 jar 文件作为 “app.jar” 添加到容器的
ENTRYPOINT 执行项目 app.jar。为了缩短 Tomcat 启动时间，添加一个系统属性指向 “/dev/./urandom” 作为 Entropy Source

如果是第一次打包，它会自动下载java 8的镜像作为基础镜像，以后再制作镜像的时候就不会再下载了。

### 4.将jar包拷贝到和Dockerfile同文件夹
在服务器新建一个docker文件夹，将maven打包好的jar包和Dockerfile文件复制到服务器的docker文件夹下

### 5.制作镜像
```
docker build -t springbootdemo4docker .
```

### 6.运行镜像
```
docker run -d -p 8088:8088 --name springbootdemo4docker
```


