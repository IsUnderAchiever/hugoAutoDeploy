---
title: 搭建Gitee图床
description: 搭建Gitee图床
date: 2023-07-09
slug: 搭建Gitee图床
image: 202412211443704.png
categories:
    - 系统配置
---

# 搭建Gitee图床
我个人常用的markdown应用有typora、vscode和息流三款
1. typora收费，网络上有不少破解教程
2. vscode可以安装插件来写markdown文档，需要知道markdown的语法格式
3. 息流使用起来相对更加简单，有免费空间限制，如果只是编写文档也足够了
在使用typora编写文档时，图片存储是个问题，虽然可以调整为相对路径存储，但是还需要保存好对应的本地图片，不然不知道图片被删掉了的话...

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012120337.avif)

相对路径存储

这个时候可以搭建一个图床，需要先下载名为【PicGo】的工具
github下载地址 https://github.com/Molunerfinn/PicGo
123网盘下载地址 https://www.123pan.com/s/tMU0Vv-31zUd.html
安装好PicGo默认即可，之后需要创建一个Gitee仓库

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121381.avif)

新建仓库

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012122963.avif)

创建私有仓库

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121608.avif)
个人主页

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121208.avif)

个人设置

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121102.avif)

私人令牌

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121096.avif)

生成新令牌

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012122665.avif)

提交信息

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012122065.avif)

复制令牌

**这里很重要了，最好是保存一下生成的这串令牌，后续无法再查看到了**
接下来开始配置PicGo

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121970.avif)

安装插件

安装一下【gitee】插件

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012120865.avif)

配置Gitee信息

上方的repo填写如下图片的地址

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121380.avif)

仓库地址

token则填写生成的【私人令牌】
**接下来别忘了开源仓库**

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012120855.avif)

进入私有仓库

找到仓库后进入

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121203.avif)

管理

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121432.avif)

设置仓库

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012121081.avif)

添加readme文件

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012120070.avif)

再次进入管理

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012122254.avif)

开源Gitee仓库

最后配置一下Typora的文件上传

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012120802.avif)

配置Typora

为了避免上传的图片重名，可以打开PicGo的【时间戳重命名】配置

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012122554.avif)

配置时间戳

![img](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202307012122013.avif)

typora上传图片结果

最后提一嘴，我使用过阿里云OSS和Github的图床，阿里云OSS倒是没什么问题，可能会有少许收费，Github懂得都懂，连接不太好毕竟是在国外，可能会造成图片传不上去，或者传上去了显示不出来
