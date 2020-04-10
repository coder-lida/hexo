title: 微服务网关 Spring Cloud Gateway
tags:
  - 微服务
categories:
  - Dev
  - Frame
date: 2020-01-01 11:09:00
cover: true

---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/weifuwu.jepg)
<!-- more -->
## 什么是网关
假设你现在要做一个电商应用，前端是移动端的APP，后端是各种微服务。那你可能某个页面需要调用多个服务的数据来展示。如果没有网关，你的系统看起来就是这个样子的：
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01YTQ4NGE3ZTU4Y2M4ODk3LnBuZw?x-oss-process=image/format,png)

而如果加上了网关，你的系统就会变成这个样子：
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS02MzQxNWM5YzY0YzViYWVmLnBuZw?x-oss-process=image/format,png)
#Spring Cloud Gateway
Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Netflix Zuul，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。
### 相关概念
* Route（路由）：这是网关的基本构建块。它由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配。
* Predicate（断言）：这是一个 Java 8 的 Predicate。输入类型是一个 ServerWebExchange。我们可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。
* Filter（过滤器）：这是org.springframework.cloud.gateway.filter.GatewayFilter的实例，我们可以使用它修改请求和响应。
### 工作流程
![gateway.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1jNTlmNTA3MTU0NDBkNjdhLnBuZw?x-oss-process=image/format,png)

（PS：看到这张图是不是很熟悉，没错，很像SpringMVC的请求处理过程）

* 请求发送到网关，DispatcherHandler是HTTP请求的中央分发器，接管请求并将请求匹配到相应的 HandlerMapping。

* 请求与处理器之间有一个映射关系，网关将会对请求进行路由，handler 此处会匹配到 RoutePredicateHandlerMapping，匹配请求对应的 Route。

* 随后到达网关的 web 处理器，该 WebHandler 代理了一系列网关过滤器和全局过滤器的实例，如对请求或者响应的 Header 处理（增加或者移除某个 Header）。

* 最后，转发到具体的代理服务。

简而言之：
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1hMzVmNzhiZTMzNjJhN2EwLnBuZw?x-oss-process=image/format,png)
客户端向 Spring Cloud Gateway 发出请求。如果 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。 过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。
## 快速开始
### 1.新建一个项目gatewayTest
在项目中添加3个module`eureka,producer,gateway`
项目结构
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0zMWY0NWYzNWNhMjg2MTc0LnBuZw?x-oss-process=image/format,png)
### 2.rureka
新建module

![step1.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00YzI0YjFlZWYwMjg3ZDEwLnBuZw?x-oss-process=image/format,png)
![step2.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1mNzMwMjIwY2ZhMGM2NjUzLnBuZw?x-oss-process=image/format,png)
![step3.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1iYjc4NmVmZjgxMGIwY2NjLnBuZw?x-oss-process=image/format,png)
![step4.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01ZWU4NDdkZGZjOWNmY2M2LnBuZw?x-oss-process=image/format,png)
![step5.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xZGQ0YTllM2RkOGFmZGE4LnBuZw?x-oss-process=image/format,png)

添加eureka依赖
```
 <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
 </dependency>
```
完整pom
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.eureka</groupId>
    <artifactId>eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>eureka</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>com.gateway.test</groupId>
        <artifactId>gatewayTest</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>..</relativePath> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
配置文件
```
spring:
  application:
    name: eureka

server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    fetch-registry: false
    register-with-eureka: false
    service-url:
      defaultZone: http://localhost:8761/eureka/

```
启动类
```
package com.example.eureka.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}

```
启动程序，访问http://localhost:8761/![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS05NzY5NzhkMzZmOGM1NmJlLnBuZw?x-oss-process=image/format,png)
现在还没有服务进行注册
### 3.producer
新建producer的module，同创建rureka，不同处如下图，其他都一样。
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01ZjY1YTg2Y2ZmZTM3NTI4LnBuZw?x-oss-process=image/format,png)

完整pom
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.producer</groupId>
    <artifactId>producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>producer</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>com.gateway.test</groupId>
        <artifactId>gatewayTest</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>..</relativePath> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
配置文件
```
spring:
  application:
    name: producer
server:
  port: 8081

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```
启动类
```
package com.example.producer.producer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableEurekaClient
@SpringBootApplication
public class ProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }

}

```
新建2个类控制器
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xMjk5MjM2Y2Y5MzQyOTMwLnBuZw?x-oss-process=image/format,png)
`HelloController`
```
@RestController
@RequestMapping("/hello")
public class HelloController {

    @RequestMapping("say")
    public String say() {
        return "Hello Every Buddy";
    }
}
```
`GoodByeController`
```
@RestController
@RequestMapping("/goodbye")
public class GoodByeController {

    @RequestMapping("say")
    public String say() {
        return "Bye Bye";
    }
}
```

启动程序，访问http://localhost:8761/
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS02NzU2N2M5NTNiM2VkMDIyLnBuZw?x-oss-process=image/format,png)

### 4.gateway
创建过程同eureka
完整pom
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.gateway</groupId>
    <artifactId>gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>gateway</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>com.gateway.test</groupId>
        <artifactId>gatewayTest</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>..</relativePath> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```
配置文件
```
test:
  uri: lb://producer

spring:
  application:
    name: gateway
#  cloud:
#    gateway:
#      routes:
#        - id: route_producer_hello
#          uri: ${test.uri} # uri以lb://开头（lb代表从注册中心获取服务），后面接的就是你需要转发到的服务名称
#          predicates:
#            - Path=/api-hello/**
#          filters:
#            - StripPrefix=1 # 表示在转发时去掉api
#
#        - id: route_producer_goodbye
#          uri: ${test.uri}
#          predicates:
#            - Path=/api-goodbye/**
#          filters:
#            - StripPrefix=1
#            - name: Hystrix
#              args:
#                name: myfallbackcmd
#                fallbackUri: forward:/user/fallback


server:
  port: 8080

logging:
  level:
    org.springframework.cloud.gateway: TRACE
    org.springframework.http.server.reactive: DEBUG
    org.springframework.web.reactive: DEBUG
    reactor.ipc.netty: DEBUG
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    enabled: true # 是否启用注册服务 默认为true, false是不启用
  instance:
    prefer-ip-address: true

```
启动类
```
package com.example.gateway.gateway;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class GatewayApplication {

    @Value("${test.uri}")
    private String uri;

    @Bean
    public RouteLocator routeLocator(RouteLocatorBuilder builder){
        return builder.routes()
                .route(r ->r.path("/hello/**").uri(uri))
                .route(r ->r.path("/goodbye/**").uri(uri)).build();
    }

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}

```
启动程序，访问http://localhost:8761/
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xODVjYTViMDA1MDI4YmU5LnBuZw?x-oss-process=image/format,png)
### 5.测试
服务都已经注册到reureka,我们定义了hello和goodbye开头的请求都会转发到`lb://producer`服务，我们定义gateway的端口是8080，producer的端口是8081
`直接请求producer服务`
http://localhost:8081/hello/say
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS04MzJmN2YxMGQ2MTZmN2U1LnBuZw?x-oss-process=image/format,png)
http://localhost:8081/goodbye/say
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0wYTcyNTA0MzBmODU1MzY0LnBuZw?x-oss-process=image/format,png)

`通过网关请求`
http://localhost:8080/hello/say
![hello.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0wYzI3OTZlNWZjZDM0ZTFmLnBuZw?x-oss-process=image/format,png)
http://localhost:8080/goodbye/say
![goodbye.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xZGJhN2QwYTBjM2M2MGVhLnBuZw?x-oss-process=image/format,png)

## 网关本身的负载均衡

那所有微服务就只有一个网关，万一并发量上去了，网关承受不住怎么办？
Spring Cloud Gateway底层是Netty的，它本身就能承受比较大的并发。如果还是承受不了并发量，那可以注册多个Gateway实例，然后在前面弄一个Nginx或者F5等负载均衡器。大概图是这样：
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01NjRlNTZkYjE2MzU4MjBkLnBuZw?x-oss-process=image/format,png)











