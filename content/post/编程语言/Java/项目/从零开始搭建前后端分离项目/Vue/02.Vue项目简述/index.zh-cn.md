---
title: 从零开始搭建前后端分离项目02
description: 从零开始搭建前后端分离项目02
date: 2022-12-11 22:00:00
slug: 从零开始搭建前后端分离项目02
image: 202412211946837.png
categories:
    - Vue
---
1. ![12](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201945625.png)
   编译App.vue之后，通过这个index.html来显示
2. access是存放静态资源的地方，比如图片、css、js
3. ![13](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201945165.png)
   component是存放组件的地方
   如图，HelloWorld.vue是一个组件，通过export导出后在其他vue文件中引用
   比如某宝等购物平台，页面的头部和底部基本不变，所以可以做成组件后在其他vue文件中直接引用
4. ![14](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201945002.png)
   index.js【index.ts】是写路由的地方
   实际上是将url路径和页面进行映射，通过url路径来访问页面
   比如：
   输入localhost:8080/ 会显示HomeView.vue的内容
   输入localhost:8080/about 会显示AboutView.vue的内容
5. ![15](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201945465.png)
   store则是用来定义页面的一些变量
   比如，登录之后的用户信息、页面之间跳转的时候需要携带的参数等都可以放在store内
6. ![16](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201945707.png)
   view是存放视图的地方，上图中就引入了HelloWorld组件
   1. 在components中定义HelloWorld组件
   2. 然后通过import的方式进行导入
   3. 最后可以通过\<HelloWorld/>这样的标签进行使用
   图中msg可以在HelloWorld中找到
   ![17](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946562.png)
   ![18](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946226.png)
   图中HomeView中将msg传入HelloWorld中，然后通过胡子语法来使用，即{{msg}}
   ![19](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202301201946701.png)
