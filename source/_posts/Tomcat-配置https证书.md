title: Tomcat 配置https证书
tags:
  - Tomcat
categories:
  - Dev
  - Container
date: 2020-03-02 17:54:00
cover: true

---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/tomcat.jpg)
<!-- more -->
>TTPS 是安全套接字层超文本传输协议，在http 的基础上加入了 SSL协议，需要使用证书来校验身份。 HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。其默认端口为：443。越来越多的网站使用了https，这里简介其相关配置。
## 一、使用jdk创建证书
### 1、keytool的概念
　keytool 是个密钥和证书管理工具。它使用户能够管理自己的公钥/私钥对及相关证书，用于（通过数字签名）自我认证（用户向别的用户/服务认证自己）或数据完整性以及认证服务。在JDK 1.4以后的版本中都包含了这一工具，它的位置为`%JAVA_HOME%\bin\keytool.exe`，如下图所示：
 
![keytool.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/tomcat-1.png)

### 2、keytool的用法
这里在安装有 JDK 环境的情况下进行，利用 keytool 工具生成 tomcat 证书，可使用 --help 命令查看相关残数据说明，具体如下：
```
Microsoft Windows [版本 6.1.7601]
版权所有 (c) 2009 Microsoft Corporation。保留所有权利。

C:\Users\Administrator>keytool
密钥和证书管理工具

命令:

 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令

使用 "keytool -command_name -help" 获取 command_name 的用法

C:\Users\Administrator>keytool -genkeypair --help
keytool -genkeypair [OPTION]...

生成密钥对

选项:

 -alias <alias>                  要处理的条目的别名
 -keyalg <keyalg>                密钥算法名称
 -keysize <keysize>              密钥位大小
 -sigalg <sigalg>                签名算法名称
 -destalias <destalias>          目标别名
 -dname <dname>                  唯一判别名
 -startdate <startdate>          证书有效期开始日期/时间
 -ext <value>                    X.509 扩展
 -validity <valDays>             有效天数
 -keypass <arg>                  密钥口令
 -keystore <keystore>            密钥库名称
 -storepass <arg>                密钥库口令
 -storetype <storetype>          密钥库类型
 -providername <providername>    提供方名称
 -providerclass <providerclass>  提供方类名
 -providerarg <arg>              提供方参数
 -providerpath <pathlist>        提供方类路径
 -v                              详细输出
 -protected                      通过受保护的机制的口令

使用 "keytool -help" 获取所有可用命令
```
### 3、创建证书
这里使用如下命令生成证书：
```
keytool -genkeypair -alias "tomcat" -keyalg "RSA" -keystore "E:\tomcat.keystore"  
```
>参数说明：
-genkeypair 表示生成密钥对;
-alias 表示别名，该别名是公开的;
-keyalg 表示密钥算法名称,本例中的采用通用的RAS加密算法;
-keystore 表示密钥库的路径及名称，不指定的话，默认在操作系统的用户目录下生成一个".keystore"的文件

然后安装提示输入，但需要注意的是 `名字`与`姓氏` 那个是填写`域名`的。 此外，尽量前后密码一致，避免后面出现密码错误的问题。密钥库的密码至少必须6个字符，可以是纯数字或者字母或者数字和字母的组合等等,其他的可以不填。
```
C:\Users\Administrator>keytool -genkeypair -alias "tomcat" -keyalg "RSA" -keysto
re "E:\tomcat.keystore"
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  local.test.com
您的组织单位名称是什么?
  [Unknown]:
您的组织名称是什么?
  [Unknown]:
您所在的城市或区域名称是什么?
  [Unknown]:
您所在的省/市/自治区名称是什么?
  [Unknown]:
该单位的双字母国家/地区代码是什么?
  [Unknown]:
CN=local.test.com, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown是否正
确?
  [否]:  y

输入 <tomcat> 的密钥口令
        (如果和密钥库口令相同, 按回车):
```

![keytool.png](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/tomcat-2.png)

这里随便写了个域名，是一个不存在的二级域名，为了能够访问，需要在 C:\Windows\System32\drivers\etc 路径下的 hosts 文件添加：
```
127.0.0.1     local.test.com 
```
## 二、tomcat 配置 https 证书
通过上面的步骤，生成了密钥，修改tomcat配置文件：/conf/server.xml, 添加如下设置：
```
<Connector 
     port="443" 
     protocol="org.apache.coyote.http11.Http11Protocol" 
     maxThreads="150" 
     SSLEnabled="true" 
     scheme="https" 
     secure="true" 
     clientAuth="false" 
     sslProtocol="TLS" 
     keystoreFile="E:\tomcat.keystore" 
     keystorePass="test8544903" />
```
其中，keystoreFile 指明密钥位置，keystorePass 指明密钥密码。
在该配置文件中有 https的 Connector 配置，被注释掉了。https 默认端口为 443。
上面介绍了自己生成密钥配置https的方法，只适合本地测试.
