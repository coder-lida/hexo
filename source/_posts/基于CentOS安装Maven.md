title: 基于CentOS安装Maven
tags:
  - Maven
categories: 
  - Dev
  - Tools
date: 2020-03-28 15:05:00
cover: true
---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/maven.jpg)
<!-- more -->
## 下载

### 1.通过官网下载
Maven官网：[http://maven.apache.org/](http://maven.apache.org/)

Maven下载地址：[http://maven.apache.org/download.cgi](http://maven.apache.org/download.cgi)

将下载好的包通过ftp上传到服务器。

### 2.wget下载
这里使用了[华中科技大学开源镜像站](http://mirrors.hust.edu.cn/),网上有很多，自行选择。
```
[root@localhost local]# wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
--2020-03-28 09:04:47--  http://mirrors.hust.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
正在解析主机 mirrors.hust.edu.cn (mirrors.hust.edu.cn)... 202.114.18.160
正在连接 mirrors.hust.edu.cn (mirrors.hust.edu.cn)|202.114.18.160|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：8842660 (8.4M) [application/octet-stream]
正在保存至: “apache-maven-3.5.4-bin.tar.gz”

100%[=======================================================================================================================================================================================================>] 8,842,660   5.30MB/s 用时 1.6s   

2020-03-28 09:04:49 (5.30 MB/s) - 已保存 “apache-maven-3.5.4-bin.tar.gz” [8842660/8842660])

```

## 解压
```
tar zxf apache-maven-3.5.4-bin.tar.gz
```

## 配置环境变量
```
cd /etc
ll
vi profile
#按i进入编辑状态
#添加maven的环境变量
export M2_HOME=/usr/local/apache-maven-3.5.4
export PATH=$PATH:$M2_HOME/bin
#编辑完成按Esc退出编辑状态，然后按:wq保存退出。
#保存退出后运行下面的命令使配置生效
source /etc/profile
```
验证
```
[root@localhost etc]# mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-18T02:33:14+08:00)
Maven home: /usr/local/apache-maven-3.5.4
Java version: 1.8.0_241, vendor: Oracle Corporation, runtime: /usr/local/jdk1.8.0_241/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-957.el7.x86_64", arch: "amd64", family: "unix"

```
配置成功！