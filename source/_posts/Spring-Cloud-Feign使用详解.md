---
title: Spring Cloud Feign使用详解
tags:
  - SpringCloud
categories:
  - 技术
date: 2019-11-19 08:35:00
cover: true

---
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00NWUyZTA5MmY3MGVmYzY2LmpwZw?x-oss-process=image/format,png)
<!-- more -->

## 背景
Feign是一个声明式的Web Service客户端，它的目的就是让Web Service调用更加简单。Feign提供了HTTP请求的模板，通过编写简单的接口和插入注解，就可以定义好HTTP请求的参数、格式、地址等信息。
而Feign则会完全代理HTTP请求，我们只需要像调用方法一样调用它就可以完成服务请求及相关处理。Feign整合了Ribbon和Hystrix(关于Hystrix我们后面再讲)，可以让我们不再需要显式地使用这两个组件。

## 特点
总起来说，Feign具有如下特性：

* 可插拔的注解支持，包括Feign注解和JAX-RS注解;

* 支持可插拔的HTTP编码器和解码器;

* 支持Hystrix和它的Fallback;

* 支持Ribbon的负载均衡;

* 支持HTTP请求和响应的压缩。

## FeignClient注解的一些属性
![](/asset/2019-11-19/p15.png)
## 快速开始
### 1.引入Feign
```
     <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-openfeign</artifactId>
     </dependency>
```
### 2.@FeignClient注解
SpringbootApplication启动类加上@FeignClient注解
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class ServiceFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceFeignApplication.class, args);
    }

}
//@EnableDiscoveryClient 和@EnableEurekaClient是等效的
```
### 3.yaml配置
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8765
spring:
  application:
    name: service-feign
```
### 4.Feign的使用
feign接口
```
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {

    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam("name") String name);
}
```
访问接口
```
@RestController
public class HiController {

    @Autowired
    SchedualServiceHi schedualServiceHi;

    @GetMapping(value = "/hi")
    public String sayHi(@RequestParam String name){
        return schedualServiceHi.sayHiFromClientOne(name);

    }
}
```
## 实战
### 1.创建项目
创建一个Maven项目feign，新建2个module `service-hi`和`service-feign`
### 2.service-hi
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0yNDg5OTI4ZWIzMDA1OTZiLnBuZw?x-oss-process=image/format,png)

pom
```
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```
yml
```
server:
  port: 8763

spring:
  application:
    name: service-hi

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
ServiceHiApplication
```
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@RestController
public class ServiceHiApplication {

    public static void main(String[] args) {
        SpringApplication.run( ServiceHiApplication.class, args );
    }

    @Value("${server.port}")
    String port;

    @RequestMapping("/hi")
    public String home(@RequestParam(value = "name", defaultValue = "forezp") String name) {
        return "hi " + name + " ,i am from port:" + port;
    }

}
```
### 3.service-feign
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1jMTVkN2JkNjUwZDQzYzZmLnBuZw?x-oss-process=image/format,png)
pom
```
 <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>
```
yml
```
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8765
spring:
  application:
    name: service-feign
```
ServiceFeignApplication
```
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableFeignClients
public class ServiceFeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceFeignApplication.class, args);
    }

}
```
SchedualServiceHi
```
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {

    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam("name") String name);
}
```
HiController
```
@RestController
public class HiController {

    @Autowired
    SchedualServiceHi schedualServiceHi;

    @GetMapping(value = "/hi")
    public String sayHi(@RequestParam String name){
        return schedualServiceHi.sayHiFromClientOne(name);

    }
}

```
## 测试
启动eureka
访问http://localhost:8761/
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1hZDk3Y2M1MmExYzU2MzQ4LnBuZw?x-oss-process=image/format,png)
启动service-hi
刷新http://localhost:8761/
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00ODI5MzE5NWZhN2RhMjZhLnBuZw?x-oss-process=image/format,png)
访问service-hihttp://localhost:8763/hi?name=kobe
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1lZjQ2NzE5MDFiYjgwNjg4LnBuZw?x-oss-process=image/format,png)
启动service-feign
刷新http://localhost:8761/
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1mZmM1NTE0ZDZhZGU5ZGZhLnBuZw?x-oss-process=image/format,png)
同过feign访问接口http://localhost:8765/hi?name=KG
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1mNWM4NGUzMDZhMDRmY2MyLnBuZw?x-oss-process=image/format,png)
可以看到去到了service-hi服务，端口8763
