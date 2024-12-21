---
title: 添加"通过XXX打开"的右键菜单
description: 添加"通过XXX打开"的右键菜单
date: 2023-03-27
slug: 添加"通过XXX打开"的右键菜单
image: pawel-czerwinski-8uZPynIu-rQ-unsplash_hu3425483315149503896.jpg
categories:
    - 系统配置
---

# 添加【通过XXX打开】的右键菜单

> 推介使用`ContextMenuManager`进行管理，以下不再演示`ContextMenuManager`用法
> [参考博文](https://www.cnblogs.com/cjxstart/p/16130101.html)
>
> 暂时还没有`以webstorm`打开的快捷方式
> win+R，输入`regedit`

![步骤3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303261149268.png)

> 新建项，命名为webstorm

[步骤4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303261151163.png)

> 点击`默认项`，输入`Open Folder as WebStorm Project`

![步骤5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303261153982.png)

> 设置图标

![新建Icon字符串值](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303261200992.png)

> 命名为Icon
> 赋值webStorm文件路径
> 设置之前新建的Icon的值

![image-20230326115803952](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202303261158016.png)

> 参考`"D:\IntelliJ IDEA 2022.3.1\bin\idea64.exe" "%1"`这个命令，将文件路径换成自己的，其他不变
