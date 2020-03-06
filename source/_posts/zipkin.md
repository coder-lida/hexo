title: zipkin
tags:
  - 微服务
categories:
  - 技术
date: 2020-01-01 11:39:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/img%20%2820%29.jpeg)
<!-- more -->
### Waht is zipkin?
![zipkin.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0zYjQ1NTgzNDJhNWQ3NWVlLnBuZw?x-oss-process=image/format,png)
Zipkin是一种分布式跟踪系统。它有助于收集解决微服务架构中的延迟问题所需的时序数据。它管理这些数据的收集和查找。Zipkin的设计基于Google Dapper论文。

应用程序用于向Zipkin报告时序数据。Zipkin UI还提供了一个依赖关系图，显示了每个应用程序通过的跟踪请求数。如果要解决延迟问题或错误，可以根据应用程序，跟踪长度，注释或时间戳对所有跟踪进行筛选或排序。选择跟踪后，您可以看到每个跨度所需的总跟踪时间百分比，从而可以识别问题应用程序。
### 快速开始
下面我们将逐步构建并启动Zipkin实例，以便在本地检查Zipkin。有三个选项：使用Java，Docker或从源代码运行。

如果您熟悉Docker，这是首选的方法。如果您不熟悉Docker，请尝试通过Java或源代码运行。
>无论您如何启动Zipkin，请浏览http：// your_host：9411以查找跟踪！
#### Docker
[Docker zipkin](https://github.com/openzipkin/docker-zipkin)工程可以创建docker 镜像, 提供脚本和一个[docker-compose.yml](https://github.com/openzipkin/docker-zipkin/blob/master/docker-compose.yml) 用于启动预建的镜像。最快的开始是直接运行最新的镜像：
```
docker run -d -p 9411:9411 openzipkin/zipkin
```
#### Java
如果安装了Java 8或更高版本，最快的方法是获得最新版本后，通过java启动
```
>curl -sSL https://zipkin.io/quickstart.sh  |  bash -s

>java -jar zipkin.jar
```
#### Running from Source（源代码运行）
Zipkin可以从源代码运行。要实现这一点，您需要获得[zipkin源码](https://github.com/openzipkin/zipkin)
>get the latest source

>git clone [https://github.com/openzipkin/zipkin](https://github.com/openzipkin/zipkin)

>cd zipkin

>Build the server and also make its dependencies

>./mvnw -DskipTests --also-make -pl zipkin-server clean install

>Run the server

>java -jar ./zipkin-server/target/zipkin-server-*exec.jar

