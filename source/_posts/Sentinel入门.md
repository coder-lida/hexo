title: Sentinel入门
tags:
  - Sentinel
  - 微服务
categories:
  - Dev
  - Frame
date: 2020-04-01 15:39:00
cover: true

---
![](http://q6pznk9ej.bkt.clouddn.com/sentinel.png)
<!-- more -->
## 前言

### Sentinel 是什么？
随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

### Sentinel 的历史

*   2012 年，Sentinel 诞生，主要功能为入口流量控制。
*   2013-2017 年，Sentinel 在阿里巴巴集团内部迅速发展，成为基础技术模块，覆盖了所有的核心场景。Sentinel 也因此积累了大量的流量归整场景以及生产实践。
*   2018 年，Sentinel 开源，并持续演进。
*   2019 年，Sentinel 朝着多语言扩展的方向不断探索，推出 [C++ 原生版本](https://github.com/alibaba/sentinel-cpp)，同时针对 Service Mesh 场景也推出了 [Envoy 集群流量控制支持](https://github.com/alibaba/Sentinel/tree/master/sentinel-cluster/sentinel-cluster-server-envoy-rls)，以解决 Service Mesh 架构下多语言限流的问题。
*   2020 年，推出 [Sentinel Go 版本](https://github.com/alibaba/sentinel-golang)，继续朝着云原生方向演进。


### Sentinel 特征

* 丰富的应用场景：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
* 完备的实时监控：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
* 广泛的开源生态：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
* 完善的 SPI 扩展点：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

### Sentinel 的主要特性
![图片.png](http://q6rnahf7l.bkt.clouddn.com/Sentinel%20%E7%9A%84%E4%B8%BB%E8%A6%81%E7%89%B9%E6%80%A7.png)

### Sentinel 的开源生态
![图片.png](http://q6rnahf7l.bkt.clouddn.com/Sentinel%20%E7%9A%84%E5%BC%80%E6%BA%90%E7%94%9F%E6%80%81.png)

### Sentinel 分为两个部分

* 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
* 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

## 快速开始
### 1.添加pom依赖
```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
    <version>1.7.1</version>
</dependency>
```
>注意: 从 Sentinel 1.5.0 开始仅支持 JDK 1.7 或者以上版本。Sentinel 1.5.0 之前的版本最低支持 JDK 1.6。

## 2.定义资源
接下来，我们把需要控制流量的代码用 Sentinel API SphU.entry("HelloWorld") 和 entry.exit() 包围起来即可。
```
public static void main(String[] args) {
    // 配置规则.
    initFlowRules();
    while (true) {
        Entry entry = null;
        try {
	    entry = SphU.entry("HelloWorld");
            /*您的业务逻辑 - 开始*/
            System.out.println("hello world");
            /*您的业务逻辑 - 结束*/
	} catch (BlockException e1) {
            /*流控逻辑处理 - 开始*/
	    System.out.println("block!");
            /*流控逻辑处理 - 结束*/
	} finally {
	   if (entry != null) {
	       entry.exit();
	   }
	}
    }
}
```
### 3.定义规则
接下来，通过规则来指定允许该资源通过的请求次数，例如下面的代码定义了资源 HelloWorld 每秒最多只能通过 20 个请求。
```
private static void initFlowRules(){
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule();
    rule.setResource("HelloWorld");
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // Set limit QPS to 20.
    rule.setCount(20);
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```
## 注解支持
Sentinel 提供了 `@SentinelResource` 注解用于定义资源，并提供了 AspectJ 的扩展用于自动定义资源、处理 `BlockException` 等。使用 [Sentinel Annotation AspectJ Extension](https://github.com/alibaba/Sentinel/tree/master/sentinel-extension/sentinel-annotation-aspectj) 的时候需要引入以下依赖：
```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
    <version>x.y.z</version>
</dependency>
```
将 SentinelResourceAspect 注册为一个 Spring Bean
```
@Configuration
public class SentinelAspectConfiguration {

    @Bean
    public SentinelResourceAspect sentinelResourceAspect() {
        return new SentinelResourceAspect();
    }
}
```
示例代码
```
public class TestService {

    // 对应的 `handleException` 函数需要位于 `ExceptionUtil` 类中，并且必须为 static 函数.
    @SentinelResource(value = "test", blockHandler = "handleException", blockHandlerClass = {ExceptionUtil.class})
    public void test() {
        System.out.println("Test");
    }

    // 原函数
    @SentinelResource(value = "hello", blockHandler = "exceptionHandler", fallback = "helloFallback")
    public String hello(long s) {
        return String.format("Hello at %d", s);
    }
    
    // Fallback 函数，函数签名与原函数一致或加一个 Throwable 类型的参数.
    public String helloFallback(long s) {
        return String.format("Halooooo %d", s);
    }

    // Block 异常处理函数，参数最后多一个 BlockException，其余与原函数一致.
    public String exceptionHandler(long s, BlockException ex) {
        // Do some log here.
        ex.printStackTrace();
        return "Oops, error occurred at " + s;
    }
}
```

## 代码实战
新建一个SpringBoot的项目
### 1.pom依赖
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.example.sentinel</groupId>
    <artifactId>sentinel-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sentinel-demo</name>
    <description>sentinel demo</description>

    <properties>
        <java.version>1.8</java.version>
        <spring.boot.version>2.2.1.RELEASE</spring.boot.version>
        <sentinel.version>1.7.0</sentinel.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-core</artifactId>
            <version>${sentinel.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-annotation-aspectj</artifactId>
            <version>${sentinel.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-transport-simple-http</artifactId>
            <version>${sentinel.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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

### 2.Controller
```
package com.example.sentinel.sentineldemo.controller;

import com.alibaba.csp.sentinel.Entry;
import com.alibaba.csp.sentinel.SphU;
import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.example.sentinel.sentineldemo.service.TestSentinelService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * @Author: Monday
 * @Date: 2020/4/1 0001 上午 11:44
 * @Description:
 */
@Controller
@RequestMapping("test")
public class TestSentinelController {

    private static final String KEY = "queryOne";

    @Autowired
    private TestSentinelService testSentinelService;

    /**
     * 代码不加任何限流 熔断
     *
     * @return
     */
    @RequestMapping("/getValue_0")
    @ResponseBody
    @SentinelResource("queryZero")
    public String getValue_0(@RequestParam("key") String key) {
        return testSentinelService.getValue_0(key);
    }


    /**
     * 限流实现方式一: 抛出异常的方式定义资源
     *
     * @param key
     * @return
     */
    @RequestMapping("/getValue_1")
    @ResponseBody
    public String getValue_1(@RequestParam("key") String key) {
        Entry entry = null;
        // 资源名
        String resourceName = KEY;
        try {
            // entry可以理解成入口登记
            entry = SphU.entry(resourceName);
            // 被保护的逻辑, 这里为查询接口
            return testSentinelService.getValue_1(key);
        } catch (BlockException blockException) {
            // 接口被限流的时候, 会进入到这里
            return "接口限流, 返回空";
        } finally {
            // SphU.entry(xxx) 需要与 entry.exit() 成对出现,否则会导致调用链记录异常
            if (entry != null) {
                entry.exit();
            }
        }
    }

    /**
     * 限流实现方式二: 注解定义资源
     *
     * @param key
     * @return
     */
    @RequestMapping("/getValue_2")
    @ResponseBody
    public String getValue_2(@RequestParam("key") String key) {
        String res = testSentinelService.getValue_2(key);
        return res;
    }

}

```
### 3.Service
```
package com.example.sentinel.sentineldemo.service;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.alibaba.csp.sentinel.slots.block.RuleConstant;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRule;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRuleManager;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.util.ArrayList;
import java.util.List;

/**
 * @Author: Monday
 * @Date: 2020/4/1 0001 上午 11:45
 * @Description: 商品查询接口
 */
@Component
@Slf4j
public class TestSentinelService {

    private static final String KEY = "queryTwo";

    /**
     * 代码不加任何限流 熔断
     *
     * @param key
     * @return
     */
    public String getValue_0(String key) {
        System.out.println("获取Value:" + key);
        return "return value :" + key;
    }


    /**
     * 抛出异常的方式定义资源
     *
     * @param key
     * @return
     */
    public String getValue_1(String key) {
        System.out.println("获取Value:" + key);
        return "return value :" + key;
    }

    /**
     * 注解定义资源
     *
     * @param key
     * @return
     */
    @SentinelResource(value = KEY, blockHandler = "blockHandlerMethod", fallback = "queryFallback")
    public String getValue_2(String key) {
        // 模拟调用服务出现异常
        if ("0".equals(key)) {
            throw new RuntimeException();
        }
        return "query value success, " + key;
    }

    public String blockHandlerMethod(String key, BlockException e) {
        return "queryValue error, blockHandlerMethod res: " + key;
    }

    public String queryFallback(String key, Throwable e) {
        return "queryValue error, return fallback res: " + key;
    }

    /**
     * 初始化限流配置
     */
    @PostConstruct
    public void initDegradeRule() {
        List<FlowRule> rules = new ArrayList<FlowRule>();
        FlowRule rule1 = new FlowRule();
        rule1.setResource(KEY);
        // QPS控制在2以内
        rule1.setCount(2);
        // QPS限流
        rule1.setGrade(RuleConstant.FLOW_GRADE_QPS);
        rule1.setLimitApp("default");
        rules.add(rule1);
        FlowRuleManager.loadRules(rules);
    }
}

```
### 4.控制台
#### 4.1下载
从 [release 页面](https://github.com/alibaba/Sentinel/releases) 下载截止目前为止最新版本的控制台 jar 包
![图片.png](http://q6rnahf7l.bkt.clouddn.com/sentinel-release.png)
>注意： 
启动 Sentinel 控制台需要 JDK 版本为 1.8 及以上版本
  从 Sentinel 1.6.0 起，Sentinel 控制台引入基本的 登录 功能，默认用户名和密码都是 sentinel

用户可以通过如下参数进行配置
* `-Dsentinel.dashboard.auth.username=sentinel` 用于指定控制台的登录用户名为 sentinel
* `-Dsentinel.dashboard.auth.password=123456` 用于指定控制台的登录密码为 123456；如果省略这两个参数，默认用户和密码均为 sentinel
* `-Dserver.servlet.session.timeout=7200` 用于指定 Spring Boot 服务端 session 的过期时间，如 7200 表示 7200 秒；60m 表示 60 分钟，默认为 30 分钟

#### 4.2启动
```
java -jar sentinel-dashboard-1.7.1.jar
```
访问http://localhost:8080
![图片.png](http://q6rnahf7l.bkt.clouddn.com/sentinel-login.png)
#### 4.3登录
![图片.png](http://q6rnahf7l.bkt.clouddn.com/sentinel-console.png)
可以看到当前控制台中没有任何的应用，因为还没有应用接入。

### 5.客户端接入
启动了控制台模块后，控制台页面都是空的，需要接入客户端。
#### 5.1导入与控制台通信的jar包
```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>${sentinel.version}</version>
</dependency>
```
#### 5.2 配置应用启动参数
引入了依赖之后，接着就是在我们的应用中配置 JVM 启动参数，如下所示：
```
-Dproject.name=xxx -Dcsp.sentinel.dashboard.server=consoleIp:port

```
其中的consoleIp和port对应的就是我们部署的 sentinel dashboard 的ip和port，我这里对应的是 127.0.0.1 和 8080，按照实际情况来配置 dashboard 的ip和port就好了，如下图所示：
![图片.png](http://q6rnahf7l.bkt.clouddn.com/sentinel-vm-config.png)

#### 5.3 启动应用
启动上边的springboot项目

#### 5.4 测试效果
本demo中http://localhost:8083/test/getValue_2?key=kobe接口执行多次，会触发限流操作，这时候再去看控制台：
![图片.png](http://q6rnahf7l.bkt.clouddn.com/sentinel-sonsole-1.png)

![图片.png](http://q6rnahf7l.bkt.clouddn.com/sentinel-console-2.png)

![图片.png](http://q6rnahf7l.bkt.clouddn.com/sentinel-console-3.png)

详细说明请参考 [官方 Sentinel Wiki](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2FSentinel%2Fwiki)









