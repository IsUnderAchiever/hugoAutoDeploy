---
title: 自定义异常未被@RestControllerAdvice捕获
description: 自定义异常未被@RestControllerAdvice捕获
date: 2023-04-05
slug: 自定义异常未被@RestControllerAdvice捕获
image: 202412212133331.png
categories:
    - Java
---

自定义异常未被捕获
=========
自定义异常未被@RestControllerAdvice捕获
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
> 自定义了一个异常未被`@RestControllerAdvice`捕获
>
> 原因是被try..catch..捕获了，而没被`@RestControllerAdvice`捕获
>
> 这个原因其实我之前听说过但是没遇到过，现在遇到了，真的长记性了…
![image-20230405130313221](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152329730.png)
