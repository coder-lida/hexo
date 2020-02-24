---
title: Spring Boot 两种部署到服务器的方式
tags:
  - 打包部署
categories:
  - Java
date: 2019-11-19 08:46:00
cover: true

---

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0zOTkzMjE4ZjcwZmU0YzNjLmpwZw?x-oss-process=image/format,png)
<!-- more -->

## jar包(官方推荐)

jar包方式启动，也就是使用spring boot内置的tomcat运行。服务器上面只要你配置了jdk1.8及以上，就ok。不需要外置tomcat 

### 1.打成jar包

### 2.将jar包放到任意目录
执行下面的命令
```
$ nohup java -jar test.jar >temp.txt &

//这种方法会把日志文件输入到你指定的文件中，没有则会自动创建。进程会在后台运行。
```
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS04NTNjYTcxMDAzNWQ4ZTg3LnBuZw?x-oss-process=image/format,png )
### 3.放开端口
阿里云服务器需要放开对应的端口
添加安全组：我的项目中配置的启动端口是18080，故这里需要放开18080端口，才能访问 

## war包
传统的部署方式：将项目打成war包，放入tomcat 的webapps目录下面，启动tomcat，即可访问。

开发环境：jdk1.8 + IDEA

下面搭建一个demo演示如何打war包部署并且如何访问：spring boot + maven

### 1.新建项目
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1kNmUzY2EwZjYyOTkyOTkzLnBuZw?x-oss-process=image/format,png )
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00YjhiMWRkYTIyYjU0ZDUyLnBuZw?x-oss-process=image/format,png )
这里我们默认打成jar包，不用修改。

### 2.修改启动Application文件
项目新建完成后，修改启动Application文件继承SpringBootServletInitializer,实现configure方法 
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01NjA3MDQ3ZTI4ZTY3ZGI2LnBuZw?x-oss-process=image/format,png )
```
@SpringBootApplication
@RestController
public class Demo1Application extends SpringBootServletInitializer {

    // 用来测试访问
    @RequestMapping("/")
    public String home() {
        return "hello 朋友";
    }

    public static void main(String[] args) {
        SpringApplication.run(Demo1Application.class, args);
    }

    // 继承SpringBootServletInitializer 实现configure 方便打war 外部服务器部署。
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Demo1Application.class);
    }
}

```
### 3.修改pom.xml
```
<packaging>war</packaging>
```
完整pom.xml代码如下
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>demo1</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <!-- 这里打成war包 若打jar，需将war改为jar -->
    <packaging>war</packaging>

    <name>demo1</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>demo1</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>

```
### 4.打包

这里可以直接到项目根目录下面：运行 maven package命令，打包。

我这里直接使用idea打包，如下图： 
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1jMWIzMDgwMTdhMWE5YWViLnBuZw?x-oss-process=image/format,png)
5.将war放入外部tomcat的webapps目录下 
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0zMWZkZjIzZWYwODRjNmM2LnBuZw?x-oss-process=image/format,png)
6.启动tomcat 
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS04Y2M1ZDFkMTZhNTljZDQ4LnBuZw?x-oss-process=image/format,png )
## 小结 

### 1.对比两种打包方式
jar更加简单，方便。具体使用哪种方式，应视应用场景而定。

### 2.注意
再说一次，将项目打成war包，部署到外部的tomcat中，这个时候，不能直接访问spring boot 项目中配置文件配置的端口。application.yml中配置的server.port配置的是spring boot内置的tomcat的端口号, 打成war包部署在独立的tomcat上之后, 你配置的server.port是不起作用的。一定要注意这一点！！
其实我们从tomcat的启动界面，已经可以看出，是启动的哪个端口： 
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0wNGQ3YzI2YmY0NTRiNDAzLnBuZw?x-oss-process=image/format,png )
很明显，日志告诉我们，我们应该访问8080端口。
下图是使用spring boot 内置tomcat启动日志，可以看出配置的server.port是生效了的！
