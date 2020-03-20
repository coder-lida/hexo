title: 什么是 开发环境、测试环境、生产环境、UAT环境、仿真环境
tags:
  - 规范
categories: 
  - Dev
  - Java
date: 2020-03-19 15:00:00
cover: true

---

![](http://q6pznk9ej.bkt.clouddn.com/car.jpeg)
<!-- more -->
**开发环境**：开发环境是程序猿们专门用于开发的服务器，配置可以比较随意， 为了开发调试方便，一般打开全部错误报告。

**测试环境**：一般是克隆一份生产环境的配置，一个程序在测试环境工作不正常，那么肯定不能把它发布到生产机上。

**生产环境**：是指正式提供对外服务的，一般会关掉错误报告，打开错误日志。可以理解为包含所有的功能的环境，任何项目所使用的环境都以这个为基础，然后根据客户的个性化需求来做调整或者修改。

三个环境也可以说是系统开发的三个阶段：开发->测试->上线，其中生产环境也就是通常说的真实环境。

**UAT环境**：UAT，(User Acceptance Test),用户接受度测试 即验收测试，所以UAT环境主要是用来作为客户体验的环境。

**仿真环境**：顾名思义是和真正使用的环境一样的环境（即已经出售给客户的系统所在环境，也成为商用环境），所有的配置，页面展示等都应该和商家正在使用的一样，差别只在环境的性能方面。

系统内部[集成测试](https://www.baidu.com/s?wd=%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YLnWnLnAcsmWTvmHKbrHmv0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPHnLnHTznHDL)(System Integration Testing) SIT
用户[验收测试](https://www.baidu.com/s?wd=%E9%AA%8C%E6%94%B6%E6%B5%8B%E8%AF%95&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YLnWnLnAcsmWTvmHKbrHmv0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPHnLnHTznHDL)(User Acceptance Testing) UAT
SIT在前，UAT在后，UAT测完才可以上线。

SIT是[集成测试](https://www.baidu.com/s?wd=%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YLnWnLnAcsmWTvmHKbrHmv0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPHnLnHTznHDL)UAT是[验收测试](https://www.baidu.com/s?wd=%E9%AA%8C%E6%94%B6%E6%B5%8B%E8%AF%95&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YLnWnLnAcsmWTvmHKbrHmv0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPHnLnHTznHDL)从时间上看，UAT要在SIT后面，[UAT测试](https://www.baidu.com/s?wd=UAT%E6%B5%8B%E8%AF%95&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YLnWnLnAcsmWTvmHKbrHmv0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPHnLnHTznHDL)要在[系统测试](https://www.baidu.com/s?wd=%E7%B3%BB%E7%BB%9F%E6%B5%8B%E8%AF%95&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YLnWnLnAcsmWTvmHKbrHmv0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPHnLnHTznHDL)完成后才开始。从测试人员看，SIT由公司的测试员来测试，而UAT一般是由用户来测试。
