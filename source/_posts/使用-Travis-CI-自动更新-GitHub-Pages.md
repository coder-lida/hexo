title: 使用 Travis CI 自动更新 GitHub Pages
tags:
  - Hexo
categories:
  - Dev
  - Hexo
date: 2020-01-15 11:26:00
cover: true
---
![](https://cdn.jsdelivr.net/gh/coder-lida/CDN/img/hexo-1.png)
<!-- more -->
## 前言
Github Pages 不能运行动态程序，只能输出一些静态内容。因此 Github Pages 非常适合用于前端项目的展示。可用于存放项目介绍、项目文档或者个人博客。本文介绍了怎么用 Travis CI 自动化部署 Github Pages。

## CI
持续集成（Continuous integration）是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误。目前 github 开源项目用的较多的 CI 主要是 Circle CI 和 Travis CI，两者都是利用容器技术来适配不同项目环境。

## GitHub Pages
hexo虽然可以方便地部署github静态博客，但是仅仅是把最终生成的html保存在repository中，像原始的Markdown文件，hexo配置文件，主题配置文件，修改文件都仅仅是保存在本地。这样不利于保存，也无法查看每篇博客的修改历史。
在github上搭blog最大的问题就是，每次提交都需要先hexo g，然后再hexo d生成文件，这样哪怕是改一个小的地方都需要重新编译全部blog。

## Travis CI  & GitHub Pages
以我的博客为例
`coder-lida.github.io`:博客静态文件仓库
`blog-source`:文档源码仓库
文档源码放置在blog-source仓库，最终部署文件在 coder-lida.github.io仓库。当在 blog-source仓库更改某些内容之后，通过运行 hexo g来生成最终部署的 HTML 文件到 public 目录，随后再进入 public 目录初始化 git 仓库、添加文件、提交文件，最后将提交强制推送到远程coder-lida.github.io仓库。

![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01MTFmNDM0YWRjZDQ0YTJhLnBuZw?x-oss-process=image/format,png)

## Start Travis CI
### .travis.yml
[Hexo官网配置](https://hexo.io/docs/github-pages),官方配置是在同一个仓库的不同分支来实现的。我没用这种方式，而是创建了2个仓库，一个放部署的网页，另一个放blog的源文件，包括post和themes。

### 注册并设置Travis CI
[Travis CI官网](https://travis-ci.org)
点击右上角使用Github登录的按钮
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0yMjJmMzAzYzlhN2QwZjBiLnBuZw?x-oss-process=image/format,png)
登录成功后
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1mNTQwMTY4MmY3YWI5YjA1LnBuZw?x-oss-process=image/format,png)
选中博客的源文件仓库，点击Setting
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1hZjcyOTY1ODJmMTgyYmY0LnBuZw?x-oss-process=image/format,png)
### 关于GH_TOKEN
登录到GitHub，点击右上角头像框，选择Settings
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01Y2ZmNjFjMDlmMmFiMzdkLnBuZw?x-oss-process=image/format,png)
选择Developer settings
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS00ODkyY2QxNTNlMmRiOTIxLnBuZw?x-oss-process=image/format,png)
添加一个token
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS03ODQ4NTNiYjJiM2RhZjJiLnBuZw?x-oss-process=image/format,png)
填写token description，比如叫hexo deploy.
勾选上授予的权限，比如我勾选的是repo和gist，然后create.
将产生的token串复制保留下来，后面会使用到,如果丢失，只能重新产生。
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1hOTA1NWVkNmMzYzMxZDAxLnBuZw?x-oss-process=image/format,png)

将生成的token设置到Travis CI作为环境变量。
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xMzU0NTgxZGEwZDJmMDQzLnBuZw?x-oss-process=image/format,png)

到此Travis CI设置完成。
### 添加 .travis.yml
先贴出我的配置
```
language: node_js #设置语言
node_js: stable # nodejs版本
branches:
  only:
    - master #只监测master分支
cache:
    apt: true
    directories:
        - node_modules # 缓存不经常更改的内容
before_install:
  - git config --global user.name "your name"  # github用户名
  - git config --global user.email "your email"  # 邮箱
  - npm install -g hexo-cli
install:
  - npm i
before_script:
# 无其他依赖项所以执行npm run build 构建就行了
script:
  - hexo clean  #清除
  - hexo generate #生成
after_script:
  - git clone https://${GH_REF} .deploy_git  # GH_REF是最下面配置的仓库地址
  - cd .deploy_git
  - git checkout master
  - cd ../
  - mv .deploy_git/.git/ ./public/   # 这一步之前的操作是为了保留master分支的提交记录，不然每次git init的话只有1条commit
  - cd ./public
  - git init
  - git add .
  - git commit -m "Travis CI Auto Builder at `date +"%Y-%m-%d %H:%M"`" # 提交记录包含时间 
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master #GH_TOKEN是在Travis中配置环境变量的名称
env:
 global:
   - GH_REF: github.com/coder-lida/coder-lida.github.io.git #设置GH_REF,换成自己用户名和仓库名
```

哪些文件需要提交到源文件的仓库
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS01OTU3YTcxOGFiNDg5ZmQ0LnBuZw?x-oss-process=image/format,png)

`package-lock.json`文件在`.gitignore`文件中加上
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0yODgzYmRmYjhiMmVlM2I4LnBuZw?x-oss-process=image/format,png)

在经历的11次失败之后，第12次终于成功了。

![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS05MDI3ZGVmMzM3Y2RhYWI0LnBuZw?x-oss-process=image/format,png)
![图片.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS0xZTBjMjc4Yjg4MzNkMTg5LnBuZw?x-oss-process=image/format,png)

