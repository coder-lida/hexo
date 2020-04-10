title: '快速搭建 Git 服务器[Windows版]'
tags:
  - Git
categories:
  - Dev
  - Tools
date: 2020-03-25 13:57:00
cover: true

---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/git.jpg)
<!-- more -->
## 服务器搭建

### 下载
* 下载 JDK：[https://www.oracle.com/technetwork/java/javase/downloads/](https://www.oracle.com/technetwork/java/javase/downloads/)
* 下载 Gitblit：[http://gitblit.com/](http://gitblit.com/)
![图片.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/gitblit-4.png)

### 解压
解压缩下载的压缩包即可，无需安装。
![图片.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/gitblit-5.png)

### 创建本地存储文件夹
![图片.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/gitblit-3.png)

### 配置
打开data文件夹下的`gitblit.properties`
在第17行可以看到
```
include = defaults.properties
```
同文件夹下找到`defaults.properties`
将上边配置的本地存储文件夹的路径复制过来
```
#git.repositoriesFolder = ${baseFolder}/git
git.repositoriesFolder = E:\GitBlit\Repository
```
找到server.httpPort，设定http协议的端口号
```
# Standard http port to serve.  <= 0 disables this connector.
# On Unix/Linux systems, ports < 1024 require root permissions.
# Recommended value: 80 or 8080
#
# SINCE 0.5.0
# RESTART REQUIRED
server.httpPort = 1024
```
找到server.httpBindInterface，设定服务器的IP地址。这里就设定你的服务器IP。
```
# Specify the interface for Jetty to bind the standard connector.
# You may specify an ip or an empty value to bind to all interfaces.
# Specifying localhost will result in Gitblit ONLY listening to requests to
# localhost.
#
# SINCE 0.5.0
# RESTART REQUIRED
server.httpBindInterface = localhost
```

找到server.httpsBindInterface，设定为localhost
```
# Specify the interface for Jetty to bind the secure connector.
# You may specify an ip or an empty value to bind to all interfaces.
# Specifying localhost will result in Gitblit ONLY listening to requests to
# localhost.
#
# SINCE 0.5.0
# RESTART REQUIRED
server.httpsBindInterface = localhost
```

### 运行
运行gitblit.cmd
![图片.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/gitblit-1.png)
如上图则运行成功
在浏览器中打开,现在就可以使用GitBlit了。默认用户名密码都是 admin
![图片.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/gitblit-2.png)

