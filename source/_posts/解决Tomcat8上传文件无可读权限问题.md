title: 解决Tomcat8上传文件无可读权限问题
tags:
  - Tomcat
categories:
  - Dev
  - Container
date: 2020-01-01 11:34:00
cover: true

---

![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/tomcat.jepg)
<!-- more -->
### 描述
使用springmvc做了一个文件上传的功能，上传到nginx目录下的一个文件夹，但是通过目录访问的时候却报403的错误
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00YTE5NGE0OTNmOTdmYzkxLnBuZw?x-oss-process=image/format,png)
去服务器查看了一下文件的权限，发现没有可读权限，于是定位了问题，上传的文件全都没有可读权限。

### 为什么没有可读权限
网上查阅资料发现，linux默认umask为022，对应权限为755，其它用户可读可执行。可以`vim /etc/profile`，搜索umusk关键字查看
```
if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
    umask 002
else
    umask 022
```
而tomcat8默认umask为027，对应权限为750，也就是说其它用户连可读的权限都没有。
可打开catalina.sh文件，搜索umask查看。
```
# Set UMASK unless it has been overridden
if [ -z "$UMASK" ]; then
    UMASK="0027"
fi
umask $UMASK
```
在catalina.sh文件的开篇可以看到
```
#   UMASK           (Optional) Override Tomcat's default UMASK of 0027
```
于是问题有了答案
登录到服务器，进入到tomcat的bin目录下
```
vim catalina.sh
输入i,进入编辑模式，将umask改为0022
：wq(保存退出)
```
可以看到
![3b0b9d5dc0f2d2115073293aeee4331.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00ZjU1N2JlN2UzMTVlMzE0LnBuZw?x-oss-process=image/format,png)
接下来重启tomcat，重新上传图片即可香油可读权限。


