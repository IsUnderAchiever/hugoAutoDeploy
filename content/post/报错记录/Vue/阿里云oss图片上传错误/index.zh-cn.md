---
title: 阿里云oss图片上传错误
description: 阿里云OSS上传图片遇到403这个错误（谷粒商城）
date: 2023-07-16
slug: 阿里云oss图片上传错误
image: 202412211946837.png
categories:
    - Vue
---

## 阿里云OSS上传图片遇到403这个错误（谷粒商城）
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201956093.png)
```error
upload.js?c0e8:599          POST http://gulimall-twh.oss-cn-shanghai.aliyuncs.com/ 403 (Forbidden)
```
挺纳闷的，跟着视频一起做的，没有漏掉授权类的步骤
上网查基本都是授权的问题，还有一个同志是电脑时间的问题，说是需要电脑时间必须和北京时间同步，不能有错误
但是以上都不是我报错的原因，我看了一下请求
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201957058.png)
这里**accessKeyId**是一个undefined
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201957390.png)
但是确确实实可以获取到accessKeyId
开始查看前端代码（upload组件是课件里直接发的），我猜想是upload组件这里出问题了
果不其然！！！
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201957876.png)
将accessid改为accessId即可
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201957978.png)
由于阿里云的文档改了一点，视频里是accessid，所以视频里没报错
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201957894.png)
估计是阿里云文档里后来把accessid改成accessId了
