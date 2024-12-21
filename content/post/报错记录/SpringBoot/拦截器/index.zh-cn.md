---
title: 拦截器
description: 拦截器
date: 2023-03-14
slug: 拦截器
image: 202412212133331.png
categories:
    - Java
---

# 拦截器
> 问题起因：明明已将放行过测试的路径了，页面也能正常获取到数据，拦截器里的sout却依然有两次输出
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303132201695.png" alt="拦截器请求参数" style="zoom:80%;" />
> 输出了一下请求路径，发现如下
<img src="https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303132202244.png" alt="拦截器" style="zoom:80%;" />
[参考博客](https://www.cnblogs.com/fnlingnzb-learner/p/16476760.html)
> 至于为什么会请求error，应该是没有找到favicon.ico的请求
`解决方法如下`
![image-20230313220847762](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303132208874.png)
> 1. 放过favicon.ico请求
> 2. static里增加一个图标即可
