---
title: server.port启动端口不生效
description: server.port启动端口不生效
date: 2024-04-11
slug: server.port启动端口不生效
image: 202412212133331.png
categories:
    - Java
---

# server.port启动端口不生效
![image-20240411062803925](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411062803925.png)
> 配置里写的端口是`8088`，然而启动的依然是`8080`
首先清理一下缓存
![image-20240411062859444](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411062859444.png)
> 发现没什么用
>
> maven clean后重新编译
>
> 依然没用
>
> **转properties，有用**
>
> `我这里是装过了插件`，可能是由于插件的影响
![image-20240411062957291](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411062957291.png)
![image-20240411063051701](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/image-20240411063051701.png)
