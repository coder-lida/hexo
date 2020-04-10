title: '快速搭建 Git 服务器[Linux版]'
tags:
  - Git
categories:
  - Dev
  - Tools
date: 2020-03-27 15:25:00
cover: true

---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/git.jpg)
<!-- more -->
## 下载
如果未安装wget,则先安装wget
```
yum install wget
```
安装完成
```
[root@localhost local]# yum install wget
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * epel: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
正在解决依赖关系
--> 正在检查事务
---> 软件包 wget.x86_64.0.1.14-18.el7_6.1 将被 安装
--> 解决依赖关系完成

依赖关系解决

=================================================================================================================================================================================================================================================
 Package                                                架构                                                     版本                                                               源                                                      大小
=================================================================================================================================================================================================================================================
正在安装:
 wget                                                   x86_64                                                   1.14-18.el7_6.1                                                    base                                                   547 k

事务概要
=================================================================================================================================================================================================================================================
安装  1 软件包

总下载量：547 k
安装大小：2.0 M
Is this ok [y/d/N]: y
Downloading packages:
wget-1.14-18.el7_6.1.x86_64.rpm                                                                                                                                                                                           | 547 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : wget-1.14-18.el7_6.1.x86_64                                                                                                                                                                                                  1/1 
  验证中      : wget-1.14-18.el7_6.1.x86_64                                                                                                                                                                                                  1/1 

已安装:
  wget.x86_64 0:1.14-18.el7_6.1                                                                                                                                                                                                                  

完毕！

```
下载gitblit
```
wget http://dl.bintray.com/gitblit/releases/gitblit-1.8.0.tar.gz
```
下载完成
```
[root@localhost local]# wget http://dl.bintray.com/gitblit/releases/gitblit-1.8.0.tar.gz
--2020-03-27 11:59:22--  http://dl.bintray.com/gitblit/releases/gitblit-1.8.0.tar.gz
正在解析主机 dl.bintray.com (dl.bintray.com)... 52.41.180.114, 54.191.3.105
正在连接 dl.bintray.com (dl.bintray.com)|52.41.180.114|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 302 
位置：http://d29vzk4ow07wi7.cloudfront.net/d23f30c1fe7d28648d682f387f9a16bfd05cd000da418489d00f04e10279776f?response-content-disposition=attachment%3Bfilename%3D%22gitblit-1.8.0.tar.gz%22&Policy=eyJTdGF0ZW1lbnQiOiBbeyJSZXNvdXJjZSI6Imh0dHAqOi8vZDI5dnprNG93MDd3aTcuY2xvdWRmcm9udC5uZXQvZDIzZjMwYzFmZTdkMjg2NDhkNjgyZjM4N2Y5YTE2YmZkMDVjZDAwMGRhNDE4NDg5ZDAwZjA0ZTEwMjc5Nzc2Zj9yZXNwb25zZS1jb250ZW50LWRpc3Bvc2l0aW9uPWF0dGFjaG1lbnQlM0JmaWxlbmFtZSUzRCUyMmdpdGJsaXQtMS44LjAudGFyLmd6JTIyIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNTg1MjgyMjgzfSwiSXBBZGRyZXNzIjp7IkFXUzpTb3VyY2VJcCI6IjAuMC4wLjAvMCJ9fX1dfQ__&Signature=kLEsE2~0a-gSiDvvEPDNqjAuOO8ab7-aqqzuZjDm2sRBZGtPmrkGINTxHEJn~-2hGeQkxX61okj5uV2sq92xSnkPXxSuw9WKJvRPYB35HLdXUTEj2aMbNtKV8J-Dq3eSkQEnLWv7SBOAFn07nrHJE8PpuIy0lKC~ulCXnM1WBmOvr6AWjf3Nla0kLpdBV3HtpCTeTgPNwbCSZYHyqrFtaNI~CQCW8aHQVji-wOLYsy~wyrQ0jjywB8r~P-jSCCAzcyFH7OVqMbJuDsFl63Mw7lK4OVU9jHKKZly6M8GcZXIhqBKS-Ddz9CZ9jHhuoPo5kVhn8jxGsbKHkunv1Zs-Fw__&Key-Pair-Id=APKAIFKFWOMXM2UMTSFA [跟随至新的 URL]
--2020-03-27 11:59:24--  http://d29vzk4ow07wi7.cloudfront.net/d23f30c1fe7d28648d682f387f9a16bfd05cd000da418489d00f04e10279776f?response-content-disposition=attachment%3Bfilename%3D%22gitblit-1.8.0.tar.gz%22&Policy=eyJTdGF0ZW1lbnQiOiBbeyJSZXNvdXJjZSI6Imh0dHAqOi8vZDI5dnprNG93MDd3aTcuY2xvdWRmcm9udC5uZXQvZDIzZjMwYzFmZTdkMjg2NDhkNjgyZjM4N2Y5YTE2YmZkMDVjZDAwMGRhNDE4NDg5ZDAwZjA0ZTEwMjc5Nzc2Zj9yZXNwb25zZS1jb250ZW50LWRpc3Bvc2l0aW9uPWF0dGFjaG1lbnQlM0JmaWxlbmFtZSUzRCUyMmdpdGJsaXQtMS44LjAudGFyLmd6JTIyIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNTg1MjgyMjgzfSwiSXBBZGRyZXNzIjp7IkFXUzpTb3VyY2VJcCI6IjAuMC4wLjAvMCJ9fX1dfQ__&Signature=kLEsE2~0a-gSiDvvEPDNqjAuOO8ab7-aqqzuZjDm2sRBZGtPmrkGINTxHEJn~-2hGeQkxX61okj5uV2sq92xSnkPXxSuw9WKJvRPYB35HLdXUTEj2aMbNtKV8J-Dq3eSkQEnLWv7SBOAFn07nrHJE8PpuIy0lKC~ulCXnM1WBmOvr6AWjf3Nla0kLpdBV3HtpCTeTgPNwbCSZYHyqrFtaNI~CQCW8aHQVji-wOLYsy~wyrQ0jjywB8r~P-jSCCAzcyFH7OVqMbJuDsFl63Mw7lK4OVU9jHKKZly6M8GcZXIhqBKS-Ddz9CZ9jHhuoPo5kVhn8jxGsbKHkunv1Zs-Fw__&Key-Pair-Id=APKAIFKFWOMXM2UMTSFA
正在解析主机 d29vzk4ow07wi7.cloudfront.net (d29vzk4ow07wi7.cloudfront.net)... 13.35.127.69, 13.35.127.111, 13.35.127.37, ...
正在连接 d29vzk4ow07wi7.cloudfront.net (d29vzk4ow07wi7.cloudfront.net)|13.35.127.69|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：42063149 (40M) [application/gzip]
正在保存至: “gitblit-1.8.0.tar.gz”

100%[=======================================================================================================================================================================================================>] 42,063,149  3.57MB/s 用时 13s    

2020-03-27 11:59:38 (3.07 MB/s) - 已保存 “gitblit-1.8.0.tar.gz” [42063149/42063149])

```

## 解压
我的目录放在`/usr/local/`下
```
cd usr/local
tar -xf gitblit-1.8.0.tar.gz 
```

## 修改配置
### 1.更改端口配置
>cd gitblit-1.8.0
ll
cd data
vi defaults.properties 
设置修改编辑完成后按ESC 输入:wq 保存退出
```
[root@localhost local]# cd gitblit-1.8.0
[root@localhost gitblit-1.8.0]# ll
总用量 3680
-rwxr-xr-x. 1 root root     984 5月  15 2014 add-indexed-branch.sh
-rwxr-xr-x. 1 root root      82 4月  20 2014 authority.sh
drwxr-xr-x. 6 root root     153 3月  27 14:27 data
drwxr-xr-x. 5 root root    4096 3月  27 14:27 docs
drwxr-xr-x. 2 root root    4096 3月  27 14:27 ext
-rw-r--r--. 1 root root 3685177 6月  23 2016 gitblit.jar
-rwxr-xr-x. 1 root root      52 4月  20 2014 gitblit.sh
-rwxr-xr-x. 1 root root      59 4月  20 2014 gitblit-stop.sh
-rwxr-xr-x. 1 root root      87 4月  20 2014 install-service-centos.sh
-rwxr-xr-x. 1 root root    1249 11月 23 2015 install-service-fedora.sh
-rwxr-xr-x. 1 root root      92 4月  20 2014 install-service-ubuntu.sh
-rwxr-xr-x. 1 root root     997 2月  26 2015 java-proxy-config.sh
-rw-r--r--. 1 root root   11556 1月  18 2016 LICENSE
-rwxr-xr-x. 1 root root     599 6月  17 2014 migrate-tickets.sh
-rw-r--r--. 1 root root   12237 1月  18 2016 NOTICE
-rwxr-xr-x. 1 root root     641 6月  17 2014 reindex-tickets.sh
-rwxr-xr-x. 1 root root    1224 2月  26 2015 service-centos.sh
-rwxr-xr-x. 1 root root    1512 5月  15 2014 service-ubuntu.sh
[root@localhost gitblit-1.8.0]# cd data
[root@localhost data]# ll
总用量 88
drwxr-xr-x. 2 root root    70 3月  27 14:27 certs
-rw-r--r--. 1 root root 65818 6月  23 2016 defaults.properties
drwxr-xr-x. 2 root root    25 3月  27 14:27 git
-rw-r--r--. 1 root root   535 6月  23 2016 gitblit.properties
drwxr-xr-x. 2 root root  4096 3月  27 14:27 gitignore
drwxr-xr-x. 2 root root   274 3月  27 14:27 groovy
-rw-r--r--. 1 root root    87 6月  23 2016 projects.conf
-rw-r--r--. 1 root root    74 6月  23 2016 users.conf
[root@localhost data]# vi defaults.properties 
```

找到`server.httpPort`，设定http协议的端口号
```
# Standard http port to serve.  <= 0 disables this connector.
# On Unix/Linux systems, ports < 1024 require root permissions.
# Recommended value: 80 or 8080
#
# SINCE 0.5.0
# RESTART REQUIRED
server.httpPort = 7070
```
找到`server.httpBindInterface`，设定服务器的IP地址。这里就设定你的服务器IP。
```
# Specify the interface for Jetty to bind the standard connector.
# You may specify an ip or an empty value to bind to all interfaces.
# Specifying localhost will result in Gitblit ONLY listening to requests to
# localhost.
#
# SINCE 0.5.0
# RESTART REQUIRED
server.httpBindInterface = 192.168.1.70
```

找到`server.httpsBindInterface`，设定为本机的ip
```
# Specify the interface for Jetty to bind the secure connector.
# You may specify an ip or an empty value to bind to all interfaces.
# Specifying localhost will result in Gitblit ONLY listening to requests to
# localhost.
#
# SINCE 0.5.0
# RESTART REQUIRED
server.httpsBindInterface = 192.168.1.70
```
`server.httpsPort = 8443 ` 保持默认不用修改
### 2.改变路径配置
>vi service-centos.sh 
设置修改编辑完成后按ESC 输入:wq 保存退出
```
GITBLIT_PATH=/usr/local/gitblit-1.8.0
GITBLIT_BASE_FOLDER=/usr/local/gitblit-1.8.0/data
GITBLIT_HTTP_PORT=7070
GITBLIT_HTTPS_PORT=8443
GITBLIT_LOG=/var/log/gitblit.log
```

## 启动
### 1.jar包启动
`java  -jar gitblit.jar` 即可手动启动gitblit
当按ctrl+c或者退出终端时则该进程会关闭，服务也会关闭，因此这里必须要将该jar程序放到后台运行，这里需要对gitblit.sh文件进行修改，`vi gitblit.sh `修改成如下
```
#!/bin/bash
java -jar gitblit.jar --baseFolder data >/dev/null    &
```

### 2.服务启动
将gitblit添加为服务
```
install-service-centos.sh
service gitblit  start
```

## 访问
用户名：admin
密码：admin
![图片.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/gitblit-6.png)

## 额外依赖库
如果需要
```
yum install -y gcc-c++ curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
yum install lsof
yum install net-tools
```
>lsof -i:8888 - 查看端口
kill -9 pid - 杀死服务

## 设置防火墙
如果需要
```
firewall-cmd --zone=public --add-port=7070/tcp --permanent 开启端口
firewall-cmd --zone=public --add-port=7071/tcp --permanent 开启端口
firewall-cmd --zone=public --add-port=8443/tcp --permanent 开启端口
firewall-cmd --reload 重启防火墙后生效
```
也可以全部开启http和https端口
```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
sudo systemctl restart firewalld.service
```


