---
title: Mongodb
description: Mongodb配置
date: 2024-10-20
slug: Mongodb配置
image: 202412211349060.png
categories:
    - 系统配置
---

# Mongodb

> 下载[Mongodb](https://www.mongodb.com/try/download/community)
>
> [参考博客](https://blog.csdn.net/weixin_43405300/article/details/120017878)

![image-20241020143337760](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211346668.png)

![image-20241020143406682](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211346999.png)

![image-20241020143418155](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211346630.png)

![image-20241020143516355](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347937.png)

> 如果采用`msi`安装，则无需继续后续步骤配置为`本地服务`

![image-20241020143619456](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347850.png)

> **配置环境变量**


在`path`下新建`D:\env\Database\MongoDB\bin`

> **运行MongoDB服务**

![image-20241020143830234](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347573.png)

在`data`目录下新建`db`目录

在`db`目录下打开`cmd`执行如下命令，即可启动`mongodb`服务

```sh
mongod --dbpath D:\env\Database\MongoDB\data\db
```

在浏览器中打开如下[链接](http://localhost:27017/)`http://localhost:27017`，可以看到如下页面就表示服务启动成功

![image-20241020144202681](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347452.png)

> **新建log目录用来存放日志**

![image-20241020144432823](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347833.png)

> **配置`mongo.config`**(与 bin 目录同级)

```
dbpath=D:\env\Database\MongoDB\data\db
logpath=D:\env\Database\MongoDB\data\log\mongo.log
```

![image-20241020144625635](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347647.png)

> **使用管理员权限打开`cmd`**，切换到`mongodb/bin`目录下，并输入如下命令

```sh
mongod -dbpath "D:\env\Database\MongoDB\data\db" -logpath "D:\env\Database\MongoDB\data\log\mongo.log" -install -serviceName "MongoDB"
```

![image-20241020144904532](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347084.png)

![image-20241020144934297](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347407.png)

> 使用`navicat`连接mongodb

![image-20241020145228272](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211347440.png)

