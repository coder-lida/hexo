title: HTTP 413错误解决方法
tags:
  - Http
categories:
  - Dev
  - Java
date: 2020-02-27 11:26:00

---

<!-- more -->
这是由于上传文件过大引起的。

## 代码检查
如果是springmvc的框架，用mutipartFile上传的文件，先检查配置文件中的最大上传文件胆小。
`spring-mvc.xml`
```
<!-- 上传文件拦截，设置最大上传文件大小   10M=10*1024*1024(B)=10485760 bytes 和编码，如果这里设置过小会导致图片可能无法上传-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10485760" />
        <property name="defaultEncoding" value="UTF-8" />
    </bean>
```
查看上传的文件是否超出了最大限制，根据自己的情况进行修改。

## 如果服务器使用了nginx做反向代理。
检查Nginx的文件上传大小的配置。
方法：
修改nginx配置文件，配置客户端请求大小和缓存大小
```
vim /etc/nginx/nginx.conf
```
在http{}中输入:
```
client_max_body_size 8M;(配置请求体缓存区大小) 
 client_body_buffer_size 128k;(设置客户端请求体最大值) 
```
重启nginx
```
cd sbin
./nginx -s reload
```