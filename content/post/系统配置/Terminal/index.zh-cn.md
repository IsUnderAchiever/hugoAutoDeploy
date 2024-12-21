---
title: Terminal
description: Terminal配置
date: 2022-12-27
slug: Terminal配置
image: settings-default-shell.png
categories:
    - 系统配置
---

![QQ截图20221227224429](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028768.png)

直接右键，可以选择在终端中打开//windows terminal here

![QQ截图20221227224450](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028336.png)

![QQ截图20221227224500](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028056.png)

方便很多
这里有两种安装方法

1. 微软商店安装
2. github安装
第一种直接打开商店搜索**Windows Terminal**后安装即可
第二种点击[链接](https://github.com/microsoft/Terminal)

![QQ截图20221227225214](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028196.png)

![QQ截图20221227225309](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202028737.png)

### 配置右键打开终端
在C:\Users\用户名\AppData\Local下建立文件夹Terminal
新建一个空白的reg注册表文件，命名为terminal.reg，可以使用txt变换后缀得到
编辑terminal.reg文件，写入如下语句，这里的%USERPROFILE%可以换成绝对路径C:\\Users\\用户名
```
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT\Directory\Background\shell\wt]
@="打开终端"
"Icon"="%USERPROFILE%\\AppData\\Local\\Terminal\\terminal.ico"
[HKEY_CLASSES_ROOT\Directory\Background\shell\wt\command]
@="C:\\Users\\用户名\\AppData\\Local\\Microsoft\\WindowsApps\\wt.exe"
```
然后双击注册表文件即可
具体步骤可查看[安装+美化](https://blog.csdn.net/qq_43606462/article/details/107458314?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167215261616800188577114%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=167215261616800188577114&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-107458314-null-null.142^v68^control,201^v4^add_ask,213^v2^t3_control1&utm_term=terminal&spm=1018.2226.3001.4187)
