---
title: Oracle
description: Oracle配置
date: 2024-02-16
slug: Oracle
image: 202412211404987.png
categories:
    - 系统配置
---

**测试环境概述**
**服务器端**
操作系统：Windows Server 2008 企业版 64位
Oracle软件：Oracle 11g 64位
**客户端**
操作系统：  Windows 7 64位
图形界面工具：PL/SQL Developer14.0.5 64位
Oracle客户端：Oracle Win64_11gR2_client

第一步：**下载服务端Oracle 11g安装包**。

下载地址：
[链接](https://pan.baidu.com/s/1ewWtg-hQnB38C27sofA2jA)
提取码：qwer

官方网站下载地址：
[链接1](http://download.oracle.com/otn/nt/oracle11g/112010/win64_11gR2_client.zip)
[链接2](http://download.oracle.com/otn/nt/oracle11g/112010/win64_11gR2_database_1of2.zip)
[链接2](http://download.oracle.com/otn/nt/oracle11g/112010/win64_11gR2_database_2of2.zip)

注意：下载OTN上的这些软件，你需要一个OTN免费帐号，不过如果通过迅雷进行下载，就不用登陆OTN了。
也可自行在官方网站内下载其他Oracle客户端或图形界面工具版本：
https://www.oracle.com/database/technologies/instant-client/downloads.html


第二步：**Oracle 11g服务端安装**
1、解压已经下载的文件，将两个压缩文件包同时解压到同一个目录下，点击“确定”。如下图所示：
2、打开安装包路径，找到【setup.exe】双击安装。如下图所示：

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242247395.png)


3、安装前请确保电脑或服务器已经安装好microsoft .net framework 

3.5。**如不知如何进行安装，可参考（点击后边文字即刻跳转）：**[如何在内网环境下离线安装.NET Framework3.5](http://mp.weixin.qq.com/s?__biz=MzI1NTQyNzg3MQ==&mid=2247484749&idx=1&sn=75c14e2055ab80c49fc1892da341cd42&chksm=ea37513ddd40d82b6a93d4778712544878850630960b6b7decf27520744ebbceaf98368672a0&scene=21#wechat_redirect)。双击安装等待。弹出安装窗口，配置安全更新，如图所示，点“下一步”，提示未提供邮件地址，点“是”跳过。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248200.png)
![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248314.png)

4、安装选项配置，如图所示，点击“下一步”

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248326.png)

5、系统类配置，可根据自己需求进行选择。这里选择“桌面类”安装。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248154.png)

6、典型安装配置，可按实际情况修改安装路径，输入管理口令后点“下一点”。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248736.png)

因为是学习环境，所以口令输入比较简单，会提示密码复杂度校验提醒，安装会有如下提示，点“是”跳过即可。

7、先决条件检查，物理环境检查无问题，进度条100%，安装自动跳转到下个安装界面。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242247352.png)

8、概要配置界面预览，如下图所示，点“完成”开始安装产品，等待，大概20分钟左右。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248272.png)

安装过程中弹出“创建克隆数据库正在进行”，继续等待即可。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248643.png)

弹框，可查看“口令管理”，建议点点看看就可以了，不必要纠结，点击口令管理下的“确定”。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242248920.png)

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242249652.png)

9、数据库创建完成，如图所示，点击“关闭”。

![图片](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202306242249804.png)

到此Oracle就安装完成了。
