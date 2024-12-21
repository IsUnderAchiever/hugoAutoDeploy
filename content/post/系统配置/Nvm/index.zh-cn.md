---
title: Nvm
description: Nvm配置
date: 2023-01-19
slug: Nvm配置
image: 202412211357864.png
categories:
    - 系统配置
---

## nvm安装
[前往下载](https://github.com/coreybutler/nvm-windows/releases)
![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202037693.png)
##### 默认就好，免得配置环境
![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202037788.png)
![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202037422.png)
> 在`nvm`文件夹下找到`setting.txt`，进行如下配置
![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202037973.png)
```properties
##当前操作系统的位数（32/64）
arch: 64 
##是否需要代理
proxy: none 
##node的淘宝镜像
node_mirror: http://npm.taobao.org/mirrors/node/
##npm的淘宝镜像
npm_mirror: https://npm.taobao.org/mirrors/npm/
```
```bash
## nvm下载nodejs
nvm install 14.17.2
nvm use 14.17.2
nvm install 16.13.2
nvm use 16.13.2
```
```bash
## 安装cnpm,将npm设置为淘宝镜像
## 安装
npm install -g cnpm --registry=https://registry.npm.taobao.org
## 设置
npm config set registry https://registry.npm.taobao.org
## 安装vue/cli
cnpm install -g @vue/cli
## 检查是否安装成功
#vue -V 或者
vue --version
```
## 创建一个vue项目试试
```bash
vue create demo
```
> 选择manually select features 手动创建
> 选择`TypeScript`、`Router`、`Vuex`，**取消掉Linter/Formatter**
![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202037971.png)
> 选择3.X
![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202037983.png)
>根据提示运行命令
```bash
cd demo
npm run serve
```
![8](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301202037071.png)
## 安装nodejs
[网址](https://nodejs.org/dist/v10.16.3/)
一直默认即可（安装地址自定义）
环境变量 path新建"D:\nodejs"
在 nodejs 安装目录下，创建 “node_global” 和 “node_cache” 两个文件夹
```bash
npm config set prefix "D:\nodejs\node_global"
npm config set cache "D:\nodejs\node_cache"
```
环境变量 path新建"D:\nodejs\node_global"
```bash
npm config get registry
npm config set registry https://registry.npm.taobao.org/
npm config get registry
```
