title: SpringBoot使用Undertow代替tomcat
tags:
  - SpringBoot
categories:
  - 技术
date: 2019-08-14 15:56:00
cover: true

---
![](http://q6pznk9ej.bkt.clouddn.com/img%20%2818%29.jpeg)
<!-- more -->

>Undertow 是基于java nio的web服务器，应用比较广泛，内置提供的PathResourceManager，可以用来直接访问文件系统；如果你有文件需要对外提供访问，除了ftp,nginx等，undertow 也是一个不错的选择，作为java开发，服务搭建非常简便

## Undertow使用
### 依赖
spring boot内嵌容器默认为tomcat，想要换成undertow，非常容易，只需修改spring-boot-starter-web依赖，移除tomcat的依赖：
```
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
        <exclusions>  
            <exclusion>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-starter-tomcat</artifactId>  
            </exclusion>  
        </exclusions>  
    </dependency>  
```
然后，添加undertow依赖
```
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-undertow</artifactId>  
    </dependency>  
```
 这样即可，使用默认参数启动undertow服务器。如果需要修改undertow参数，继续往下看。

### undertow的参数设置：
```
server:  
    port: 8084  
    http2:  
        enabled: true  
    undertow:  
        io-threads: 16  
        worker-threads: 256  
        buffer-size: 1024  
        buffers-per-region: 1024  
        direct-buffers: true 
```
io-threads：IO线程数, 它主要执行非阻塞的任务，它们会负责多个连接，默认设置每个CPU核心一个线程，不可设置过大，否则启动项目会报错：打开文件数过多。

 

worker-threads：阻塞任务线程池，当执行类似servlet请求阻塞IO操作，undertow会从这个线程池中取得线程。它的值取决于系统线程执行任务的阻塞系数，默认值是 io-threads*8

 

以下配置会影响buffer，这些buffer会用于服务器连接的IO操作，有点类似netty的池化内存管理。

buffer-size：每块buffer的空间大小，越小的空间被利用越充分，不要设置太大，以免影响其他应用，合适即可

buffers-per-region：每个区分配的buffer数量，所以pool的大小是buffer-size * buffers-per-region

direct-buffers：是否分配的直接内存(NIO直接分配的堆外内存)

## File Server
```
import java.io.File;

import io.undertow.Handlers;
import io.undertow.Undertow;
import io.undertow.server.handlers.resource.PathResourceManager;

public class FileServer {
    public static void main(String[] args) {
        File file = new File("/");
        Undertow server = Undertow.builder().addHttpListener(8080, "localhost")
                .setHandler(Handlers.resource(new PathResourceManager(file.toPath(), 100))
                        .setDirectoryListingEnabled(true))
                .build();
        server.start();
    }
}
```
好了！运行main函数，打开浏览器访问 http://localhost:8080
