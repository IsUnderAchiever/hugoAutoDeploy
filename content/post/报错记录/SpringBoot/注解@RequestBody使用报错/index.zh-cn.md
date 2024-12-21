---
title: 注解@RequestBody使用报错
description: 注解@RequestBody使用报错
date: 2023-04-05
slug: 注解@RequestBody使用报错
image: 202412212133331.png
categories:
    - Java
---

注解@RequestBody使用报错
=========
@RequestBody
------------------------------------------------------
> 今天在使用requestBody注解时，出现了一些问题
![后端代码](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152328827.png)
![报错](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152328802.png)
> 不知道咋想的，传参不是这么传的
>
> 既然用的是@RequestBody注解，如果是User user，那这么写没问题，但是写的是Integer id
>
> 那应该使用如下格式的传参
![正确格式](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202402152328851.png)
