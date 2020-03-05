title: Git常用命令
tags:
  - Git
categories:
  - 技术
date: 2020-03-05 18:53:00
cover: true

---

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMjU1MzI0OS1lYWJlMjk2MGJlZGJjMmExLmpwZw?x-oss-process=image/format,png)
<!-- more -->

>相关资料：
[廖雪峰教程链接](https://www.liaoxuefeng.com/wiki/896043488029600)
[Git远程操作详解](https://www.ruanyifeng.com/blog/2014/06/git_remote.html)
[Git查看、删除、重命名远程分支和tag](https://blog.zengrong.net/post/1746.html)

## 操作流程
日常使用git更新提交代码的一般流程是这样的：
    在对代码进行了一些修改之后，使用：`git add .`或`git add -A`(`git add --all`的缩写)将本地所有新增文件添加进版本库。
    使用：git commit -m 备注 将代码提交到本地版本库。（备注内容没有空格的话不需要加引号）
    使用：git pull 从服务器拉取代码，更新本地版本库。
    使用：git push 将本地版本库推送到服务器。

>* `git add .` ：他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。
>* `git add -u` ：他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）
>* `git add -A` ：是上面两个功能的合集（git add --all的缩写）

## 创建
### 创建并切换 branch
```
git checkout -b 分支名
```
### 仅仅切换 branch
```
git checkout 分支名
```
### 创建 tag
```
git tag 标签名
```
### 创建 tag 并备注(备注信息加不加双引号都可以)
```
git tag -a 标签名 -m 备注信息
```
### 创建PGP tag 并备注
```
git tag -s 标签名 -m 备注信息
```
## 查看
### 查看本地 branch list
```
git branch 分支名
```
### 查看远程 branch list
```
git branch -r 分支名
```
### 查看所有 branch list
```
git branch -a 分支名
```
### 查看本地 tag
```
git tag
```
### 查看某个本地 tag 详情
```
git show 标签名
```
## 删除
### 删除本地 branch / tag
```
git branch -d 分支名或标签名
```
### 删除所有未推送的本地 branch
```
git fetch -p
```
### 仅仅删除某个远程 branch / tag
```
git push origin :分支名或标签名
# 或者
git push origin --delete 分支名或标签名
```
## 推送
### 推送某个 branch / tag
```
git push origin 分支名或标签名
```
### 推送所有 branch
```
git push --all origin
```
### 推送所有 tag
```
git push --tags
```
## 重命名
重命名本地分支
```
git branch -m 旧分支名 新分支名
```
重命名远程分支需要分三步操作
*  删除远程分支
* 重命名本地分支
* 推送本地分支

## 回滚
### soft （默认）
只回滚到某个commit，本地代码不变 (不加–soft或–hard默认为–soft)
```
git reset --soft 分支名或标签名
```
### hard
彻底回滚（commit和本地代码都回滚）
```
彻底回滚（commit和本地代码都回滚）
```
## 下载、合并分支
合并某本地分支到当前分支
```
git merge 分支名
```
合并某远程分支到当前分支 `直接合并，慎用`
```
git pull origin 远程分支名
```
下载某个远程标签
```
git fetch origin tag 远程标签名
```
