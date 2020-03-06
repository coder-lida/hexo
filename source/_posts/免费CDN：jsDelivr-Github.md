title: 免费CDN：jsDelivr + Github
tags:
  - Hexo
categories:
  - 技术
date: 2020-03-07 08:44:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/img%20%283%29.png)
<!-- more -->
CDN的全称是Content Delivery Network，即内容分发网络。CDN是构建在网络之上的内容分发网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN的关键技术主要有内容存储和分发技术。

放在Github的资源在国内加载速度比较慢，因此需要使用CDN加速来优化网站打开速度，jsDelivr + Github便是免费且好用的CDN，非常适合博客网站使用。

### 1、新建Github仓库 
![图片.png](http://q6rnahf7l.bkt.clouddn.com/cdn.png)

### 2、克隆Github仓库到本地 
在本地目录右键 Git Bash Here，执行以下命令：
```
git clone 一键复制的仓库地址
```
### 3、上传资源 
```
echo "# CDN" >> README.md
git init
git add README.md
git commit -m "first commit"
git push -u origin master
```
### 4、发布仓库 
点击release发布
![release.png](http://q6rnahf7l.bkt.clouddn.com/releases.png)

自定义发布版本号

![图片.png](http://q6rnahf7l.bkt.clouddn.com/releases%26tags.png)

### 5、通过jsDelivr引用资源 
使用方法：[https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@发布的版本号/文件路径](https://cdn.jsdelivr.net/gh/你的用户名/你的仓库名@发布的版本号/文件路径)

例如：https://cdn.jsdelivr.net/gh/coder-lida/CDN/css/style.css




