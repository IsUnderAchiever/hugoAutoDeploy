---
title: Nexus搭建Npm私服
description: Nexus搭建Npm私服
date: 2025-06-03
slug: Nexus搭建Npm私服
image: 202506081432952.png
categories:
    - 系统配置
---
# Nexus搭建Npm私服
> 如果希望可以自由切换npm镜像源，可以使用`nrm`

```bash
nrm ls

#   npm ---------- https://registry.npmjs.org/
#   yarn --------- https://registry.yarnpkg.com/
#   tencent ------ https://mirrors.tencent.com/npm/
#   cnpm --------- https://r.cnpmjs.org/
# * taobao ------- https://registry.npmmirror.com/
#   npmMirror ---- https://skimdb.npmjs.com/registry/
#   huawei ------- https://repo.huaweicloud.com/repository/npm/
```


![Pasted image 20250602022209.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428471.png)

这里配置了华为的proxy

![Pasted image 20250602022952.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428472.png)

新建一个group，并将proxy仓库加入组

![Pasted image 20250602023059.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428473.png)

```bash
npm config set registry http://116.148.120.214:8081/repository/npm-public/

# 新建一个vite项目并下载依赖
npm create vite@latest

cd vite-project
npm install
```

![Pasted image 20250602023142.png](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo05/202506081428474.png)

添加到nrm中

```bash
npm config get registry
# http://116.148.120.214:8081/repository/npm-public/

nrm add registry http://116.148.120.214:8081/repository/npm-public/
# SUCCESS  Add registry registry success, run nrm use registry command to use registry registry.
```