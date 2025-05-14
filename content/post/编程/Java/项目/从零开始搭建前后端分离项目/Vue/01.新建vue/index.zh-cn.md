---
title: 从零开始搭建前后端分离项目01
description: 从零开始搭建前后端分离项目01
date: 2022-12-11
slug: 从零开始搭建前后端分离项目01
image: 202412211946837.png
categories:
    - Vue
---

node环境配置略过，如果有安装多个版本的nodejs的需求，建议安装一个nvm
nvm和nodejs的安装配置在CSDN上都可以找到

![0](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201943871.png)

我这里使用nvm安装了两个版本的nodejs，正在使用的是16.13.2版本
```bash
# node版本切换回14.17.2版本
nvm user 14.17.2
# 查询可以安装的所有nodejs的版本
nvm list available
# 安装最新稳定版
nvm install stable
# 安装nodejs，version是版本号 例如：nvm install 14.17.2
nvm install <version>
# 卸载nodejs，卸载指定版本的nodejs，当安装失败时卸载使用
nvm uninstall <version>
```
## 新建vue项目
1. 新建文件夹“springboot+vue框架” 在文件夹内cmd
   vue的项目名为springboot-vue-demo
   ```bash
   vue create springboot-vue-demo
   ```
   ![1](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201945557.png)

2. 选择manually select features 手动创建

   ![2](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944496.png)

3. ![3](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201945195.png)

   按空格选择以上依赖，回车下一步

4. ![4](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944155.png)
   选择3.X

5. ![5](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944076.png)

   分别按照上述图片来输入y和n，最后一个是 选择是否保存本次的配置为模板（下次创建可直接选定这些配置），可选可不选

6. ![6](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944691.png)

   项目创建成功，可输入这两个命令来运行vue项目
   ```bash
   cd springboot-vue-demo
   npm run serve
   ```
7. 使用idea打开“springboot+vue框架”这个文件夹【根目录】
   ![7](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944612.png)
8. 在idea中配置vue项目的运行环境
   ![8](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944096.png)
9. package.json和scripts需要配置
   ![9](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944668.png)
10. 选择server后，点击“运行”
    ![10](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944321.png)
11. 在浏览器输入http://localhost:8080/ 即可访问前端项目
    ![11](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201944375.png)
