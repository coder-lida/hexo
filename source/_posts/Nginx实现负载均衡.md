title: Nginx实现负载均衡
author: 少年闰土
tags:
  - Nginx
categories:
  - Dev
  - Tools
date: 2020-04-11 13:47:00
cover: true
---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/nginx.png)
<!-- more -->

负载均衡即是代理服务器将接收的请求均衡的分发到各服务器中。

负载均衡的优势在访问量少或并发小的时候可能并不明显，且不说淘宝双11、铁道部抢票这种级别的访问量、高并发，就是一般网站的抢购活动时，也会给服务器造成很大压力，可能会造成服务器崩溃。而负载均衡可以很明显的减少甚至消除这种情况的出现，下面我们说说实现方法。

准备工作

首先下载安装Nginx。

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-1.png)

下载完成解压到本地盘符。解压后是这样的

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-2.png)

注意：nginx.exe是启动的程序，为了方便我们可以手写两个bat文件：

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-3.png)

我将nginx解压到了我本地的E盘

reload.bat

```
E:
cd kit\nginx-1.14.0\
nginx  -s reload
```

stop.bat

```
E:
cd  kit\nginx-1.14.0\
nginx -s stop
```

我们双击nginx.exe就可以启动nginx，我们启动一下，打开任务 管理器看到

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-4.png)

就说明启动成功。

接下来配置两个tomcat来进行测试，下面是我本地的tomcat，存放在E盘中。

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-5.png)

拷贝一份放到我的D盘中，并修改端口号，默认 为8080，我们将D盘中的tomcat端口号修改为8082，将E盘中的tomcat端口号修改为8081。

端口号的修改：

找到conf

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-6.png)

修改

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-7.png)

修改如下：

```
<Server port="8006" shutdown="SHUTDOWN">  
<Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />  
<Listener className="org.apache.catalina.core.JasperListener" />  
<Listener className="org.apache.catalina.mbeans.ServerLifecycleListener" />  
<Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />  
<GlobalNamingResources>    
 <Resource name="UserDatabase"
                   auth="Container"             
                   type="org.apache.catalina.UserDatabase"              
                  description="User database that can be updated and saved"              
                  factory="org.apache.catalina.users.MemoryUserDatabaseFactory"              
                 pathname="conf/tomcat-users.xml" />  
</GlobalNamingResources>  
<Service name="Catalina">    
<Connector port="8082" 
                   protocol="HTTP/1.1"                
                   connectionTimeout="20000"                
                   redirectPort="8443" />            
<Connector port="8010" 
                    protocol="AJP/1.3" 
                    redirectPort="8443" />    
<Engine name="Catalina" defaultHost="localhost">      
<Realm className="org.apache.catalina.realm.UserDatabaseRealm"             
             resourceName="UserDatabase"/>      
<Host name="localhost"  
          appBase="webapps"            
          unpackWARs="true" 
          autoDeploy="true"            
          xmlValidation="false" 
          xmlNamespaceAware="false">     
 </Host>    
</Engine>  
</Service>
</Server>
```

将E盘中的tomcat端口号修改为8081；只需修改默认文件中的一点

```
    <Connector port="8081" 
                       protocol="HTTP/1.1"               
                       connectionTimeout="20000"              
                      redirectPort="8443" />        
```

配置nginx的配置文件：

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-8.png)

找到文件中的server节点，然后再上面添加

```
upstream local_tomcat_test {          
          server localhost:8082 weight=8 max_fails=3 fail_timeout=30s;           
          server localhost:8081 weight=2 max_fails=3 fail_timeout=30s;    
 }   
```

然后修改server:

```
upstream local_tomcat_test {          
       server localhost:8082 weight=8 max_fails=3 fail_timeout=30s;          
       server localhost:8081 weight=2 max_fails=3 fail_timeout=30s;    
 }      
server {        
      listen       80;        
      server_name  localhost;        
location / {            
     proxy_pass http://local_tomcat_test;            
     #root html;           
     #index index.html index.htm       
 } 
```

配置完成后，启动两个tomcat，为了区分是哪个tomcat，我把tomcat的默认访问页进行了修改。

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-9.png)

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-10.png)

然后我们再地址栏输入localhost,试试效果：

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-11.png)

访问了tomcat-8082,刷新一下：

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/assert/nginx-12.png)

可以看到访问了tomcat-8081

各参数的含义：

```
worker_processes：工作进程个数，可配置多个
worker_connections:单个进程最大连接数
server:每一个server相当于一个代理服务器
lister:监听端口，默认80
server_name:当前服务的域名，可以有多个，用空格分隔(我们是本地所以是localhost)
location：表示匹配的路径，这时配置了/表示所有请求都被匹配到这里
index：当没有指定主页时，默认会选择这个指定的文件，可多个，空格分隔
proxy_pass：请求转向自定义的服务器列表
upstream name{ }:服务器集群名称
```

小结

nginx作为一个反向代理服务器，能缓存我们项目的静态文件，并实现反向代理与均衡负载，可以有效减少服务器压力，即使项目不大，也可以使用。

大家另外应该都还发现了个问题，虽然这样请求能分别请求到两个tomcat上，如果是一般不需身份校检的或什么认证的方法尚可，但如果出现这类情况：

我们在tomcat1上进行了登录，这时用户session当然是存在tomcat1上的，而这时进入个人中心的请求请求到tomcat2上了，这时就会出现问题了。tomcat2会告诉你还未登录，这显然不是我们想看到的。

这就涉及到session共享了，如何让两个服务器上的session共用。我这里放到下次再说，可能要过个好几天。

感谢大家支持。


